---
title: 'Vue音频实现歌词同步滚动'
date: 2020-09-25 23:41:08
tags: 'vue'
categories: '实战'
top_img: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1601058907595&di=be9d8ba9ee831f0562c7462662a3c66f&imgtype=0&src=http%3A%2F%2F00.minipic.eastday.com%2F20171009%2F20171009110449_d41d8cd98f00b204e9800998ecf8427e_6.jpeg
cover: https://cn.vuejs.org/images/logo.png
---

### 动机

​	看着网页版的qq音乐里歌词滚动，想让无聊的自己让点事情做，顺便复习一下H5的audio API

<!-- more -->
### 效果

<img src="img/image-20200925230834175.png" alt="效果图" style="zoom:50%;" />

​	这里的控制面版**controls**使用了原生的，因浏览器不同实际的**controls**也显示不同，图中为**chrome**的

### 实现

1. 建立一个audio 音频标签，src里是音频文件也可以是http网络请求

```html
<audio
  src="../mp3/shuangjiegun.mp3"
  controls="controls"
  ref="audio"
  preload="auto"
  autoplay="autoplay"
  @timeupdate="timeupdate"
 >
</audio>
```

2. 处理lrc文件，将歌词文本数据进行处理

```javascript
//methods
parseLyric(text) {
      //将文本分隔成一行一行，存入数组
      var lines = text.split("\n"),
        //用于匹配时间的正则表达式，匹配的结果类似[xx:xx.xx]
        pattern = /\[\d{2}:\d{2}.\d{2}\]/g,
        //保存最终结果的数组
        result = [];
      //去掉不含时间的行
      while (!pattern.test(lines[0])) {
        lines = lines.slice(1);
      }
      //上面用'\n'生成生成数组时，结果中最后一个为空元素，这里将去掉
      lines[lines.length - 1].length === 0 && lines.pop();
      lines.forEach(function (v /*数组元素值*/) {
        //提取出时间[xx:xx.xx]
        var time = v.match(pattern),
          //提取歌词
          value = v.replace(pattern, "");
        //因为一行里面可能有多个时间，所以time有可能是[xx:xx.xx][xx:xx.xx][xx:xx.xx]的形式，需要进一步分隔
        time.forEach(function (v1) {
          //去掉时间里的中括号得到xx:xx.xx
          var t = v1.slice(1, -1).split(":");
          //将结果压入最终数组
          result.push([parseInt(t[0], 10) * 60 + parseFloat(t[1]), value]);
        });
      });
      //最后将结果数组中的元素按时间大小排序，以便保存之后正常显示歌词
      result.sort(function (a, b) {
        return a[0] - b[0];
      });
      console.log(result);
      return result;
    }
```

3. 根据audio/video 的 **ontimeupdate**事件，在音频的播放时间段内监听当前播放的时间，判断cruentTime处于歌词的哪一个时间段中，然后控制高亮

```javascript
//methods
 timeupdate({ target, offsetY }) {
      const { currentTime } = target;
      const { lrcs, audio } = this;
      if (lrcs && lrcs.length) {
        for (let i = 0; i < lrcs.length; i++) {
          const next = lrcs[i + 1] && lrcs[i + 1][0];
          const current = lrcs[i][0];
          const last = lrcs[lrcs.length - 1][0];
          if (
            (currentTime < next && currentTime > current) || //非最后一列
            (currentTime < audio.duration && currentTime > last) //最后一列
          ) {
            // console.log(lrcs[i][1])
            this.highlighted = i;
          }
        }
      }
    }
```

4. 在页面dom加载完成后 **this.$nextTick** 中得到audio对象，并得到处理后得歌词数据

5. 通过**ontimeupdate**事件，实时更新目前歌词处于哪一段，通过得到当前段落位于所有歌词中的数组index,根据逻辑（index+1）* lineHeight 从而得到偏移量

6. 有了偏移量就能让歌词实时滚动，这里加了一段逻辑，因为要让高亮段落一直处于歌词区域的中间位置，所以对高亮index有所限制

7. 本来想要加上拖拽的，发现加上后问题多多，暂时把判断拖拽和点击的逻辑完成了

   判断条件是通过mousedown和mouseup两个事件的触发的时间差是否大于300ms，判断是否为长按和点击；

   在长按状态下，判断down时的y轴位置是否与up时相同，不同则就是拖拽操作

   通过down时的y轴位置值的相差为正负判断向上还是向下

   触发条件是使用watch观察，鼠标弹起时 up、down（位置，时间的关系）,为什么选择时mouseUp，因为up前必然先经过了down，判断时能同时拿到最新的一组数据

### 组件代码

```vue
<template>
  <div>
    <audio
      src="../mp3/shuangjiegun.mp3"
      controls="controls"
      ref="audio"
      preload="auto"
      autoplay="autoplay"
      @timeupdate="timeupdate"
    ></audio>
    <!-- <button @click="play">play</button>
    <button @click="pause">pause</button> -->
    <br /><br />
    <div id="lrcArea">
      <!-- <div id="divide" /> -->
      <div id="lyric">
        <ul :style="styleObject">
          <li
            v-for="(lrc, i) in lrcs"
            :key="i"
            :class="{ actived: i === highlighted }"
            @mouseup="mouseUp"
            @mousedown="mouseDown($event, lrc[0])"
          >
            {{ lrc[1] }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script>
import lrc from "../lrc/index";

export default {
  name: "Music",
  data() {
    return {
      audio: null,
      lrcs: null,
      highlighted: null,
      downTime: null,
      upTime: null,
      downPosition: null,
      upPosition: null,
      currlrcTime: null,
      offset:0
    };
  },
  computed: {
    styleObject() {
      const { highlighted ,offset} = this;
      if (!highlighted || highlighted < 9) {
        return { transform: `translateY(0)` };
      }
      else if(offset&&(offset/24)>=6){
        return { transform: `translateY(0)` };
      }
      return {
        transform: `translateY(${0 - (highlighted - 9) * 24}px)`,
      };
    },
  },
  created() {
    this.$nextTick(() => {
      this.audio = this.$refs.audio;
      this.lrcs = this.parseLyric(lrc.shuangJieGun);
    });
  },
  watch: {
    upTime: function (val) {
      //是否长按
      if (val - this.downTime >= 300) {
        if (this.downPosition === this.upPosition) {
          this.audio.currentTime = this.currlrcTime;
        } else {
          const direction = this.downPosition - this.upPosition > 0 //true时向上，flase向下
          const offsetY = Math.abs(this.downPosition - this.upPosition);
          const lines = Math.floor(offsetY / 24);
          this.offset = offsetY
          // if(direction){
          //    this.highlighted -= lines
          // }else {
          //   this.highlighted += lines
          // }
         
        }
      }
      //点击事件（没有长按）
      else {
        this.audio.currentTime = this.currlrcTime;
      }
    },
  },
  methods: {
    play() {
      this.audio.play();
    },
    pause() {
      this.audio.pause();
    },
    mouseUp({ timeStamp, y }) {
      this.upTime = timeStamp;
      this.upPosition = y;
    },
    mouseDown({ timeStamp, y }, value) {
      this.downTime = timeStamp;
      this.currlrcTime = value;
      this.downPosition = y;
      // console.log(timeStamp, y, "down");
    },
    timeupdate({ target, offsetY }) {
      const { currentTime } = target;
      const { lrcs, audio } = this;
      if (lrcs && lrcs.length) {
        for (let i = 0; i < lrcs.length; i++) {
          const next = lrcs[i + 1] && lrcs[i + 1][0];
          const current = lrcs[i][0];
          const last = lrcs[lrcs.length - 1][0];
          if (
            (currentTime < next && currentTime > current) || //非最后一列
            (currentTime < audio.duration && currentTime > last) //最后一列
          ) {
            // console.log(lrcs[i][1])
            this.highlighted = i;
          }
        }
      }
    },
    parseLyric(text) {
      //将文本分隔成一行一行，存入数组
      var lines = text.split("\n"),
        //用于匹配时间的正则表达式，匹配的结果类似[xx:xx.xx]
        pattern = /\[\d{2}:\d{2}.\d{2}\]/g,
        //保存最终结果的数组
        result = [];
      //去掉不含时间的行
      while (!pattern.test(lines[0])) {
        lines = lines.slice(1);
      }
      //上面用'\n'生成生成数组时，结果中最后一个为空元素，这里将去掉
      lines[lines.length - 1].length === 0 && lines.pop();
      lines.forEach(function (v /*数组元素值*/) {
        //提取出时间[xx:xx.xx]
        var time = v.match(pattern),
          //提取歌词
          value = v.replace(pattern, "");
        //因为一行里面可能有多个时间，所以time有可能是[xx:xx.xx][xx:xx.xx][xx:xx.xx]的形式，需要进一步分隔
        time.forEach(function (v1) {
          //去掉时间里的中括号得到xx:xx.xx
          var t = v1.slice(1, -1).split(":");
          //将结果压入最终数组
          result.push([parseInt(t[0], 10) * 60 + parseFloat(t[1]), value]);
        });
      });
      //最后将结果数组中的元素按时间大小排序，以便保存之后正常显示歌词
      result.sort(function (a, b) {
        return a[0] - b[0];
      });
      console.log(result);
      return result;
    },
  },
};
</script>
<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
ul {
  padding: 0;
  li {
    margin: 0;
    width: 460px;
    list-style: none;
    line-height: 1.5;
    cursor: pointer;
    user-select: none;
  }
  li.actived {
    font-size: 1.5em;
    color: red;
    -webkit-mask-image: -webkit-gradient(
      linear,
      0 0,
      0 bottom,
      from(rgba(0, 0, 255, 0)),
      to(red)
    );
  }
}
#lrcArea {
  width: 460px;
  margin: 0 auto;
  text-align: center;
  // border: 1px solid #000;
  height: 500px;
  overflow: hidden;
  #divide {
    position: absolute;
    width: 100%;
    height: 0px;
    border-top: 1px dashed #000;
    top: calc(50% + 3px);
  }
  ul {
    transition: 0.1s ease-out 0s;
  }
  &::-webkit-scrollbar {
    // display: none;
  }
}
</style>
```
### 演示地址

https://hongzhixin.github.io/VueMusicPlayer/#/

### 总结

确实需要多动了脑子，不然不会发现自己这么废柴23333 ...









