<script setup>
import { onMounted, ref } from 'vue';
// import memberData from '@/data/member.json'

const number = ref(200) // 参与抽奖数量

const member = ref([])
const selected = ref(1)
const running = ref(false)
const btns = ref([
  5, 2, 1
])
const showResult = ref(false)
const canvas = ref(null)
const result = ref([])

onMounted(() => {

  let in_canvas = document.createElement('canvas');
  in_canvas.id = 'myCanvas';
  in_canvas.width = document.body.offsetWidth;
  in_canvas.height = document.body.offsetHeight;
  document.getElementById('main').appendChild(in_canvas);

  canvas.value = in_canvas


  initNumber()
  init()
})

const  createHTML = () => {
  let choosed = JSON.parse(localStorage.getItem('choosed')) || {};
  let html = ['<ul>'];
  let getKey = function (item) {
    return item.name;
  };

  member.value.forEach(function (item) {
    let key = getKey(item);
    let color = choosed[key] ? 'yellow' : 'white';
    html.push('<li><a href="#" style="color: ' + color + ';">' + item.name + '</a></li>');
  });
  html.push('</ul>');
  return html.join('');
};

const initNumber = () => {
  // 初始化数据
  let list = []
  for (let index = 0; index < number.value; index++) {
    const element = {
      name: `No.${index + 1}`,
      value: index + 1,

    }
    list.push(element)
  }
  list.sort(() => Math.random() - 0.5);
  console.log(list.length)
  member.value = list
}

const init = () => {
  let choosed = JSON.parse(localStorage.getItem('choosed')) || {};
  console.log(choosed);
  
  
  canvas.value.innerHTML = createHTML();
  window.TagCanvas.Start('myCanvas', '', {
    textColour: null,
    initial: speed(),
    dragControl: 1,
    textHeight: 14
  });
  
}


const reset = () => {
  if (confirm('确定要重置么？所有之前的抽奖历史将被清除！')) {
    localStorage.clear();
    location.reload(true);
  }
}
const onClick = (num) => {
  showResult.value = false
  selected.value = num;
}

const speed = () => {
  return [0.1 * Math.random() + 0.01, -(0.1 * Math.random() + 0.01)];
}


const lottery = (count) => {
  let choosed = JSON.parse(localStorage.getItem('choosed')) || {};
  let getKey = function(item){
      return item.name;
  };
  let list = canvas.value.getElementsByTagName('a');
  let color = 'yellow';
  let ret = member.value
    .filter(function (m, index) {
      m.index = index
      return !choosed[getKey(m)];
    })
    .map(function (m) {
      return Object.assign({
        score: Math.random()
      }, m);
    })
    .sort(function (a, b) {
      return a.score - b.score;
    })
    .slice(0, count)
    
    ret.map(function (m) {
      choosed[getKey(m)] = 1;
      console.log(list[m.index])
      list[m.value].style.color = color;
      return m.name + '<br/>' + m.phone;
    });



  localStorage.setItem('choosed', JSON.stringify(choosed));
  return ret;
};

const toggle = () => {
  if (running.value) {
    window.TagCanvas.SetSpeed('myCanvas', speed());
    let ret = lottery(selected.value);
    if (ret.length === 0) {
      // console.log('已抽完')
      confirm('已抽完！')
      return
    }
    result.value = ret
    localStorage.setItem(new Date().toString(), JSON.stringify(ret));
    
    setTimeout(function () {
      canvas.value.innerHTML = createHTML();
      window.TagCanvas.Reload('myCanvas');
      showResult.value = true
    }, 300);
    } else {
    showResult.value = false
    window.TagCanvas.SetSpeed('myCanvas', [5, 1]);
  }
  running.value = !running.value;
}

const setMember = (e) => {
  const value = e.target.value || 200
  number.value = value
}

const setLotteryMember = () => {
  initNumber()
  init()
}

</script>

<template>
  <div>
    <main id="main" :class="{ mask: showResult }">
      <div class="tools">
        <div style="color: #fff;">
          <label>参与人数：</label>
          <input  class="pure-button" style="width: 100px;margin: 5px; text-align: left;padding-left: 8px;" v-model="number" @blur="setMember" />
          <button  class="pure-button"  @click="setLotteryMember">确定</button>
        </div>
        <div>
          <button v-for="value in btns" @click="onClick(value)" class="pure-button"
            :class="{ 'button-error': selected == value }">{{ value }}</button>
        </div>
        <div>
          <button class="pure-button" @click="toggle" :class="{
            'button-secondary': !running,
            'button-success': running
          }">{{ running ? '停!' : '开始' }}</button>
        <button class="pure-button button-warning" @click="reset">重置</button>
        </div>
      </div>
    </main>
    <div class="result" v-show="showResult">
      <span v-for="item in result">{{ item.name }}</span>
      <a @click="showResult = false">关闭</a>
    </div>
  </div>
</template>
<style lang="less" scoped>
#main {
  background-color: #121936;
  overflow: hidden;
}

.mask {
  filter: blur(5px);
}

.result {
  position: absolute;
  height: 320px;
  width: 100%;
  left: 0;
  top: 50%;
  margin-top: -160px;
  text-align: center;
  padding: 10px;

  a {
    position: absolute;
    right: 10%;
    color: red;
    cursor: pointer;
  }
}

.result span {
  display: inline-block;
  font-size: 25px;
  width: 150px;
  background: #fff;
  line-height: 30px;
  color: #000;
  margin: 5px;
  border-radius: 10px;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.8);
  padding: 10px 0;
}

button,
input,
optgroup,
select,
textarea {
  color: inherit;
  font: inherit;
  margin: 0;
  border: none;
}

button {
  overflow: visible;
}

button,
select {
  text-transform: none;
}

button,
html input[type=button],
input[type=reset],
input[type=submit] {
  -webkit-appearance: button;
  cursor: pointer;
}

.pure-button {
  display: inline-block;
  zoom: 1;
  line-height: normal;
  white-space: nowrap;
  vertical-align: middle;
  text-align: center;
  cursor: pointer;
  -webkit-user-drag: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

.pure-button {
  font-family: inherit;
  font-size: 100%;
  padding: .5em 1em;
  color: #444;
  color: rgba(0, 0, 0, .8);
  border: 0 rgba(0, 0, 0, 0);
  background-color: #E6E6E6;
  text-decoration: none;
  border-radius: 2px;
}

.pure-button:focus {
  outline: 0
}

.pure-button-hover,
.pure-button:hover,
.pure-button:focus {
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr='#00000000', endColorstr='#1a000000', GradientType=0);
  background-image: -webkit-gradient(linear, 0 0, 0 100%, from(transparent), color-stop(40%, rgba(0, 0, 0, .05)), to(rgba(0, 0, 0, .1)));
  background-image: -webkit-linear-gradient(transparent, rgba(0, 0, 0, .05) 40%, rgba(0, 0, 0, .1));
  background-image: -moz-linear-gradient(top, rgba(0, 0, 0, .05) 0, rgba(0, 0, 0, .1));
  background-image: -o-linear-gradient(transparent, rgba(0, 0, 0, .05) 40%, rgba(0, 0, 0, .1));
  background-image: linear-gradient(transparent, rgba(0, 0, 0, .05) 40%, rgba(0, 0, 0, .1));
}

.button-success,
.button-error,
.button-warning,
.button-secondary {
  color: white;
  border-radius: 4px;
  text-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
}

.button-success {
  background: rgb(28, 184, 65);
}

.button-error {
  background: rgb(202, 60, 60);
}

.button-warning {
  background: rgb(223, 117, 20);
}

.button-secondary {
  background: rgb(66, 184, 221);
}

.tools {
  position: absolute;
  bottom: 20px;
  right: 20px;
  text-align: right;
}

.tools .pure-button {
  display: inline-block;
  margin: 5px;
  padding: 10px 0;
  text-align: center;
  width: 50px;
}

.mask {
  -webkit-filter: blur(5px);
  filter: blur(5px);
}

#main {
  -webkit-transition: all 1s;
  transition: all 1s;
}

.result-btn {
  margin-top: 20px;
  text-align: right;
  margin-right: 30px;
  text-decoration: none;
  color: white;
}
</style>
