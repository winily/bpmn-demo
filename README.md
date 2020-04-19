### 前言    

&emsp; &emsp; 因为之前项目需求需要用到Activiti以及Bpmn.js，我就去学了一波操作了一波，再学习使用的过程中经常去百度发现对于Activiti7以及Bpmn.js的文章还是相对比较少的，或者说描述的不够详细以及版本相对比较旧的问题，所以现在记录一下我整合Bpmn.js的一些步骤以及操作。

&emsp; &emsp; 有关后端方面的Spring Boot整合Activiti7的一些操作我下次再整理发布一边专门的博客来记录，这次先来看一下Vue怎么样集成Bpmn.js实现在线绘图，导出Xml，svg，在线保存等操作。

&emsp; &emsp; 如果您有什么好的建议以及问题可以发送邮件到“18934086807@163.com”

&emsp; &emsp; Github 完整代码<a href="https://github.com/winily/bpmn-demo">bpmn-demo</a>
&emsp; &emsp; 作者博客<a href="http://blog.li-winily.com/">Winily</a>

### 预备工作

&emsp; &emsp; 首先不用说肯定先得有一个Vue的一个项目，先建一个Vue的项目，安装一下依赖。

```cmd  
// 创建一个Vue项目
vue create bpmn-demo 

// 安装一下项目依赖，我比较喜欢yarn，
yarn
// 或者使用npm
npm install

``` 

&emsp;&emsp;搞定项目了之后我们要做的就是安装一下Bpmn-js的依赖以及整合代码了。先安装个依赖

```cmd
// yarn 安装
yarn add bpmn-js

// npm安装
npm install bpmn-js
```

&emsp; &emsp; 首先先打开项目的main.js，导入一下有关于bpmn-js的字体库以及样式文件，如果不导入这些样式文件将会导致bpmn-js的左侧控件菜单无法显示的问题
<img src="https://user-gold-cdn.xitu.io/2020/2/22/1706aefa3d614ea0?w=813&h=590&f=png&s=22320"/>

``` JavaScript
import 'bpmn-js/dist/assets/diagram-js.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-embedded.css'
```

### 创建Bpmn建模器

&emsp; &emsp; 搞定了准备工作那就开始来在Vue中使用Bpmn-js了, 这里我为了方便直接就在App.vue中来写代码了，在实际的使用上再按照需求写到组件中去。

首先Bpmn-js的建模器其实是通过Canvas来实现的，我们在template中创建一个div供给Bpmn创建 建模器。

``` HTML
<template>
    <div id="app">
        <div class="container">
            <!-- 创建一个canvas画布 npmn-js是通过canvas实现绘图的，并设置ref让vue获取到element -->
            <div class="bpmn-canvas" ref="canvas"></div>
        </div>
    </div>
</template>
```

写好了Html部分接下来我们来实现js部分。要创建建模器我们要使用到BpmnModeler这个对象，主要通过创建这个对象创建建模器。

``` JavaScript
// 在这里引入一下Bpmn建模器对象
import BpmnModeler from "bpmn-js/lib/Modeler";

export default {
    data() {
        return {
            bpmnModeler: null,
            canvas: null,
            // 这部分具体的代码我放到了下面
            initTemplate: `略` 
        };
    },
    methods: {
        init() {
            // 获取画布 element
            this.canvas = this.$refs.canvas;
            // 创建Bpmn对象
            this.bpmnModeler = new BpmnModeler({
                // 设置bpmn的绘图容器为上门获取的画布 element
                container: this.canvas
            });

            // 初始化建模器内容
            this.initDiagram(this.initTemplate);
        },
        initDiagram(bpmn) {
            // 将xml导入Bpmn-js建模器
            this.bpmnModeler.importXML(bpmn, err => {
                if (err) {
                    this.$Message.error("打开模型出错,请确认该模型符合Bpmn2.0规范");
                }
            });
        }
    },
    // 生命周期钩子，在组件加载完成后调用init函数进行创建初始化Bpmn-js建模器
    mounted() {
        this.init();
    }
};
```

&emsp;&emsp;这部份xml就是Bpmn流程图的模板代码了，这个模板包含了一个开始节点在里面，根据需求可以按照自己需要的创建或者修改模板，当然我不推荐手动修改xml代码。创建模板的方式我推荐就是在Bpmn-js建模器中来创建一个自己想要的模型，然后导出xml或者bpmn文件再把文件里面的代码复制出来使用就好。

&emsp;&emsp;在我具体的实际应用以及跟Activiti7后端的结合中发现xml模板需要注意的一个点是xml首部，就是definitions标签部分这里引入了很多命名空间，如果你的流程图需要部署到Activiti中的话那你要注意必须要引入 <em>Activiti的命名空间</em> .
&emsp;&emsp;以下模板就是符合Activiti使用的Xml模板了。若没有引入或者不符合Activiti的规范命名空间的话那将会导致模型部署失败。如果模型最终要部署到Activiti的话建议基于以下模板修改使用。

``` XML
<?xml version="1.0" encoding="UTF-8"?>
<definitions 
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" 
  xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" 
  xmlns:camunda="http://camunda.org/schema/1.0/bpmn" 
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  xmlns:activiti="http://activiti.org/bpmn" 
  id="m1577635100724" 
  name="" 
  targetNamespace="http://www.activiti.org/testm1577635100724"
>
  <process id="process" processType="None" isClosed="false" isExecutable="true">
    <extensionElements>
      <camunda:properties>
        <camunda:property name="a" value="1" />
      </camunda:properties>
    </extensionElements>
    <startEvent id="_2" name="start" />
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_leave">
    <bpmndi:BPMNPlane id="BPMNPlane_leave" bpmnElement="leave">
      <bpmndi:BPMNShape id="BPMNShape__2" bpmnElement="_2">
        <omgdc:Bounds x="144" y="368" width="32" height="32" />
        <bpmndi:BPMNLabel>
          <omgdc:Bounds x="149" y="400" width="23" height="14" />
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

&emsp;&emsp;完成以上的步骤我们完成了建模器的创建，让我们来看看效果。

<img src="https://user-gold-cdn.xitu.io/2020/2/22/1706aefa3ce746b8?w=1920&h=915&f=png&s=24885"/>

### 添加bpmn-js-properties-panel面板插件

&emsp; &emsp;Bpmn-js原本是不支持Activities那些自定义值的设置的，需要额外引入插件进行整合。首先先要安装两个插件，分别是bpmn-js-properties-panel和camunda-bpmn-moddle

&emsp; &emsp;bpmn-js-properties-panel是给建模器提供了属性编辑器，然后camunda-bpmn-moddle就是拓展了属性编辑器可以编辑的内容，像Activitie的assignee这些属性就是要依靠camunda-bpmn-moddle来提供编辑的。

```CMD
// yarn 安装
yarn add bpmn-js-properties-panel
yarn add camunda-bpmn-moddle

// npm 安装
npm install bpmn-js-properties-panel
npm install camunda-bpmn-moddle
```

&emsp;&emsp;安装好依赖后要在main.js中引入一下bpmn-js-properties-panel的样式要不然工具栏不会显示

```JavaScript
// 左边工具栏以及编辑节点的样式
import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css'
```
&emsp;&emsp;在App.vue里要创建一个div给工具栏一个显示的位置。我直接就放canvas的下面了

```HTML
<!-- 工具栏显示的地方 -->
<div class="bpmn-js-properties-panel" id="js-properties-panel"></div>
```

&emsp;&emsp;然后要导入一下工具栏以及配置一下对工具栏的支持

```JavaScript
// 工具栏相关
import propertiesProviderModule from "bpmn-js-properties-panel/lib/provider/camunda";
import propertiesPanelModule from "bpmn-js-properties-panel";
import camundaModdleDescriptor from "camunda-bpmn-moddle/resources/camunda";
```

修改后的init函数
```JavaScript
init() {
  // 获取画布 element
  this.canvas = this.$refs.canvas;

  // 创建Bpmn对象
  this.bpmnModeler = new BpmnModeler({
    // 设置bpmn的绘图容器为上门获取的画布 element
    container: this.canvas,

    // 加入工具栏支持
    propertiesPanel: {
      parent: "#js-properties-panel"
    },
    additionalModules: [propertiesProviderModule, propertiesPanelModule],
    moddleExtensions: {
      camunda: camundaModdleDescriptor
    }
  });

  this.createNewDiagram(this.bpmnTemplate);
}
```
效果
<img src="https://user-gold-cdn.xitu.io/2020/2/22/1706aefa3d5626de?w=1037&h=907&f=png&s=57046"/>


## <center>实现新建、导入，导出操作<center>

&emsp;&emsp;实现这些功能我们先把按键和样式写上,以及给它们加上对应的点击事件, 我这里使用了Element组件

&emsp;&emsp;在这里要说明一个问题，如果你使用这个流程编辑器是为了编辑流程然后流程需要部署到Activiti来运行的话这里会有个问题，使用camunda-bpmn-moddle设置的相关属性，比如说将 Assignee设置为“张三” 最终XML出来的是 camunda:assignee="张三" 但是这个并不符合 Activiti的规范需求，在 Activiti 中需要的是 activiti:assignee="张三" 要不然无法正常识别，所以我对于这个问题的一个解决方式就是直接在 saveXML 函数中获取到 xml文本的时候直接使用正则将camunda替换成了activiti这样子流程就可以正常的运行了。

```HTML
<div class="action">
  <!-- 关于打开文件的这个我使用了Element的文件上传组件，在上传前钩子获取到文件然后读取文件内容 -->
  <el-upload class="upload-demo" :before-upload="openBpmn">
    <el-button icon="el-icon-folder-opened"></el-button>
  </el-upload>
  <el-button class="new" icon="el-icon-circle-plus" @click="newDiagram"></el-button>
  <el-button icon="el-icon-download" @click="downloadBpmn"></el-button>
  <el-button icon="el-icon-picture" @click="downloadSvg"></el-button>
</div>
```

大概的效果是这样子

<img src="https://user-gold-cdn.xitu.io/2020/2/22/1706aefa372785fd?w=809&h=902&f=png&s=18886"/>

接下来去实现操作

### 导入文件操作()

&emsp;&emsp;我使用了Element的文件上传组件，使用上传文件之前的钩子(before-upload)拿到文件对象然后从文件对象中获取到文件内容输入到Bpmn建模器。

```JavaScript
openBpmn(file) {
  const reader = new FileReader();
  // 读取File对象中的文本信息，编码格式为UTF-8
  reader.readAsText(file, "utf-8");
  reader.onload = () => {
    //读取完毕后将文本信息导入到Bpmn建模器
    this.createNewDiagram(reader.result);
  };
  return false;
}
```

### 新建图操作

&emsp;&emsp;新建图就很简单了，其实就重新加载一下最开始的模板而已
```JavaScript
newDiagram() {
  this.createNewDiagram(this.bpmnTemplate);
},
```
### 导出文件操作

&emsp;&emsp;对于导出操作我这里设置了一个隐藏的a链接，因为我的做法是获取文件内容，设置文件名，文件格式，然后生成下载链接给到a标签然后再触发a标签的点击事件来触发下载。

```HTML
<a hidden ref="downloadLink"></a>
```

下载相关操作

```JavaScript
download({ name = "diagram.bpmn", data }) {
  // 这里就获取到了之前设置的隐藏链接
  const downloadLink = this.$refs.downloadLink;
  // 把数据转换为URI，下载要用到的
  const encodedData = encodeURIComponent(data);
  if (data) {
    // 将数据给到链接
    downloadLink.href =
      "data:application/bpmn20-xml;charset=UTF-8," + encodedData;
    // 设置文件名
    downloadLink.download = name;
    // 触发点击事件开始下载
    downloadLink.click();
  }
},
```

&emsp;&emsp;为了方便呢我就写了一个获取文件名的函数，在这里我就直接拿模型ID作为文件名了。
```JavaScript
getFilename(xml) {
  let start = xml.indexOf("process");
  let filename = xml.substr(start, xml.indexOf(">"));
  filename = filename.substr(filename.indexOf("id") + 4);
  filename = filename.substr(0, filename.indexOf('"'));
  return filename;
},
```

#### 1. 导出Bpmn操作

&emsp;&emsp;对于导出Bpmn文件的操作是比较简单的，其实Bpmn文件跟XML文件没啥区别就后缀名不太一样，实际上Bpmn文件里面保存的就是XML标签结构。所以我们要导出下载Bpmn文件只要从Bpmn-js的建模器中获取到模型的XML即可。
```JavaScript
downloadBpmn() {
  this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
    if (!err) {
      // 获取文件名
      const name = `${this.getFilename(xml)}.bpmn`;
      // 将文件名以及数据交给下载函数
      this.download({ name: name, data: xml });
    }
  });
},
```

#### 2. 导出SVG操作

&emsp;&emsp;导出svg的操作就稍微麻烦了一点，因为我没找到Bpmn-js建模器有可以提供导出的函数，那我就只好自己从canvas画布中提取SVG标签出来然后再处理下载下来

```JavaScript
downloadSvg() {
  this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
    if (!err) {
      // 获取文件名
      const name = `${this.getFilename(xml)}.svg`;
      
      // 从建模器画布中提取svg图形标签
      let context = "";
      const djsGroupAll = this.$refs.canvas.querySelectorAll(".djs-group");
      for (let item of djsGroupAll) {
        context += item.innerHTML;
      }
      // 获取svg的基本数据，长宽高 
      const viewport = this.$refs.canvas
        .querySelector(".viewport")
        .getBBox();

      // 将标签和数据拼接成一个完整正常的svg图形
      const svg = `
        <svg
          xmlns="http://www.w3.org/2000/svg"
          xmlns:xlink="http://www.w3.org/1999/xlink"
          width="${viewport.width}"
          height="${viewport.height}"
          viewBox="${viewport.x} ${viewport.y} ${viewport.width} ${viewport.height}"
          version="1.1"
          >
          ${context}
        </svg>
      `;
      // 将文件名以及数据交给下载函数
      this.download({ name: name, data: svg });
    }
  });
},
```

## <center>最终效果以及完整代码<center>

效果截图
<img src="https://user-gold-cdn.xitu.io/2020/2/22/1706aefa3696a1ff?w=1920&h=915&f=png&s=76495"/>


完整代码

1. main.js

```JavaScript
import Vue from 'vue'
import App from './App.vue'
import router from './router'

// bpmn 相关依赖
import 'bpmn-js/dist/assets/diagram-js.css' 
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-embedded.css'

// 左边工具栏以及编辑节点的样式
import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css'

import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

2. App.vue

```HTML
<template>
  <div id="app">
    <div class="container">
      <!-- 创建一个canvas画布 npmn-js是通过canvas实现绘图的，并设置ref让vue获取到element -->
      <div class="bpmn-canvas" ref="canvas"></div>
      <!-- 工具栏显示的地方 -->
      <div class="bpmn-js-properties-panel" id="js-properties-panel"></div>


      <!-- 把操作按钮写在这里面 -->
      <div class="action">
        <el-upload action class="upload-demo" :before-upload="openBpmn">
          <el-button icon="el-icon-folder-opened"></el-button>
        </el-upload>
        <el-button class="new" icon="el-icon-circle-plus" @click="newDiagram"></el-button>
        <el-button icon="el-icon-download" @click="downloadBpmn"></el-button>
        <el-button icon="el-icon-picture" @click="downloadSvg"></el-button>

        <a hidden ref="downloadLink"></a>
      </div>
    </div>
  </div>
</template>

<script>
import BpmnModeler from "bpmn-js/lib/Modeler";
// 工具栏相关
import propertiesProviderModule from "bpmn-js-properties-panel/lib/provider/camunda";
import propertiesPanelModule from "bpmn-js-properties-panel";
import camundaModdleDescriptor from "camunda-bpmn-moddle/resources/camunda";

export default {
  data() {
    return {
      bpmnModeler: null,
      canvas: null,
      bpmnTemplate: `
          <?xml version="1.0" encoding="UTF-8"?>
          <definitions 
              xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" 
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
              xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" 
              xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" 
              xmlns:camunda="http://camunda.org/schema/1.0/bpmn" 
              xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
              xmlns:activiti="http://activiti.org/bpmn" 
              id="m1577635100724" 
              name="" 
              targetNamespace="http://www.activiti.org/testm1577635100724"
            >
            <process id="process" processType="None" isClosed="false" isExecutable="true">
              <extensionElements>
                <camunda:properties>
                  <camunda:property name="a" value="1" />
                </camunda:properties>
              </extensionElements>
              <startEvent id="_2" name="start" />
            </process>
            <bpmndi:BPMNDiagram id="BPMNDiagram_leave">
              <bpmndi:BPMNPlane id="BPMNPlane_leave" bpmnElement="leave">
                <bpmndi:BPMNShape id="BPMNShape__2" bpmnElement="_2">
                  <omgdc:Bounds x="144" y="368" width="32" height="32" />
                  <bpmndi:BPMNLabel>
                    <omgdc:Bounds x="149" y="400" width="23" height="14" />
                  </bpmndi:BPMNLabel>
                </bpmndi:BPMNShape>
              </bpmndi:BPMNPlane>
            </bpmndi:BPMNDiagram>
          </definitions>
      `
    };
  },
  methods: {
    newDiagram() {
      this.createNewDiagram(this.bpmnTemplate);
    },

    downloadBpmn() {
      this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
        if (!err) {
          // 获取文件名
          const name = `${this.getFilename(xml)}.bpmn`;
          // 将文件名以及数据交给下载方法
          this.download({ name: name, data: xml });
        }
      });
    },
    downloadSvg() {
      this.bpmnModeler.saveXML({ format: true }, (err, xml) => {
        if (!err) {
          // 获取文件名
          const name = `${this.getFilename(xml)}.svg`;

          // 从建模器画布中提取svg图形标签
          let context = "";
          const djsGroupAll = this.$refs.canvas.querySelectorAll(".djs-group");
          for (let item of djsGroupAll) {
            context += item.innerHTML;
          }
          // 获取svg的基本数据，长宽高
          const viewport = this.$refs.canvas
            .querySelector(".viewport")
            .getBBox();

          // 将标签和数据拼接成一个完整正常的svg图形
          const svg = `
            <svg
              xmlns="http://www.w3.org/2000/svg"
              xmlns:xlink="http://www.w3.org/1999/xlink"
              width="${viewport.width}"
              height="${viewport.height}"
              viewBox="${viewport.x} ${viewport.y} ${viewport.width} ${viewport.height}"
              version="1.1"
              >
              ${context}
            </svg>
          `;
          // 将文件名以及数据交给下载方法
          this.download({ name: name, data: svg });
        }
      });
    },

    openBpmn(file) {
      const reader = new FileReader();
      // 读取File对象中的文本信息，编码格式为UTF-8
      reader.readAsText(file, "utf-8");
      reader.onload = () => {
        //读取完毕后将文本信息导入到Bpmn建模器
        this.createNewDiagram(reader.result);
      };
      return false;
    },

    getFilename(xml) {
      let start = xml.indexOf("process");
      let filename = xml.substr(start, xml.indexOf(">"));
      filename = filename.substr(filename.indexOf("id") + 4);
      filename = filename.substr(0, filename.indexOf('"'));
      return filename;
    },

    download({ name = "diagram.bpmn", data }) {
      // 这里就获取到了之前设置的隐藏链接
      const downloadLink = this.$refs.downloadLink;
      // 把输就转换为URI，下载要用到的
      const encodedData = encodeURIComponent(data);

      if (data) {
        // 将数据给到链接
        downloadLink.href =
          "data:application/bpmn20-xml;charset=UTF-8," + encodedData;
        // 设置文件名
        downloadLink.download = name;
        // 触发点击事件开始下载
        downloadLink.click();
      }
    },

    async init() {
      // 获取画布 element
      this.canvas = this.$refs.canvas;

      // 创建Bpmn对象
      this.bpmnModeler = new BpmnModeler({
        // 设置bpmn的绘图容器为上门获取的画布 element
        container: this.canvas,

        // 加入工具栏支持
        propertiesPanel: {
          parent: "#js-properties-panel"
        },
        additionalModules: [propertiesProviderModule, propertiesPanelModule],
        moddleExtensions: {
          camunda: camundaModdleDescriptor
        }
      });

      await this.createNewDiagram(this.bpmnTemplate);
    },
    async createNewDiagram(bpmn) {
      // 将字符串转换成图显示出来;
      this.bpmnModeler.importXML(bpmn, err => {
        if (err) {
          this.$Message.error("打开模型出错,请确认该模型符合Bpmn2.0规范");
        }
      });
    }
  },
  mounted() {
    this.init();
  }
};
</script>

<style>

.bpmn-canvas {
  width: 100%;
  height: 100vh;
}

.action {
  position: fixed;
  bottom: 10px;
  left: 10px;
  display: flex;
}
.upload-demo {
  margin-right: 10px;
}

.bpmn-js-properties-panel {
  position: absolute;
  top: 0;
  right: 0px;
  width: 300px;
}
</style>
```


&emsp; &emsp; 距离上一篇的 [《Vue 整合Bpmn-js 工作流模型编辑器》](https://juejin.im/post/5e509fab6fb9a07c820fa78a) 也一个多月了，在最近又收到公司的一个需求就是要我把Bpmn.js 流程编辑器给汉化了，没得办法又要回头去汉化这个编辑器。今天呢我就跟着上篇的基础来说说怎么汉化Bpmn.js。

&emsp; &emsp; Github 完整代码<a href="https://github.com/winily/bpmn-demo">bpmn-demo</a>
&emsp; &emsp; 作者博客<a href="http://blog.li-winily.com/">Winily</a>

## 1. 先把编辑器搭建好
&emsp; &emsp; 对于这一部就直接按照我的上一篇进行搭建就好，就是上面那个《Vue 整合Bpmn-js 工作流模型编辑器》。

## 2. 接下来我们要去下载汉化文件
&emsp; &emsp; 对于这个汉化文件我是在码云上找到了一位大佬翻译好的文件，[链接](https://gitee.com/polarloves/bpmn-js/tree/master/app/customTranslate) 里面有两个文件 translationsGerman.js 和 customTranslate.js，其中 translationsGerman.js这个文件是翻译好的文本配置，customTranslate.js 的话就是汉化器了，它会按照获取到编辑器原本的配置对比 translationsGerman.js 中的配置然后进行替换操作，将相关英文替换为中文文本。

## 3. 接下来把汉化文件载入编辑器
&emsp; &emsp; 刚刚下好了那两个文件基本来说汉化操作已经完成了百分之九十了，只要进行最后的将它载入到我们的 Bpmn.js 编辑器即可。

![](https://user-gold-cdn.xitu.io/2020/3/29/17124d92ecd91bd6?w=322&h=272&f=png&s=9157)

&emsp; &emsp; 我这里为了方便就直接放在src目录下了，实际按照你们自己封装组件整理好。接下来就是在组件中导入一下然后载入即可。

![](https://user-gold-cdn.xitu.io/2020/3/29/17124df6595e8ec5?w=730&h=207&f=png&s=14318)

&emsp; &emsp; 导入后载入到编辑器中，在上一篇做好的基础上加上这两步，汉化就完成了。

![](https://user-gold-cdn.xitu.io/2020/3/29/17124e22d6bacdb7?w=625&h=549&f=png&s=22589)

## 4. 效果图

![](https://user-gold-cdn.xitu.io/2020/3/29/17124e5389a9eb00?w=1920&h=947&f=png&s=65525)