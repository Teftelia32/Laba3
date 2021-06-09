<template>
  
  <div id="app">
    <button v-on:click="to_main" type="button" class="button-1">Головна</button>
    <div v-if="status == 'main'">
      <button v-on:click="to_append_driver" type="button" class="button-2">Додати водія</button>
      <button v-on:click="to_append_driver_mark" type="button" class="button-2">Додати оцінку</button>
      <button v-on:click="to_list_drivers" type="button" class="button-2">Список водіїв</button>
      <button v-on:click="to_find_driver" type="button" class="button-2">Знайти водія</button>
      
    </div>
   <div v-else-if="status == 'append_driver'" class="row">
      <form onsubmit="return false;">
        <label for="exampleInputEmail1" class="superClass">ПІБ</label>
        <input type="text" id="exampleInputEmail1" aria-describedby="emailHelp" v-model="append_driver_data[0]" placeholder="Введіть ПІБ" ref="pib" class="form-control">

        <label for="exampleInputEmail1" class="superClass">Email</label>
        <input type="email" id="exampleInputEmail1" aria-describedby="emailHelp" v-model="append_driver_data[1]" placeholder="Введіть Email" ref="pib" class="form-control">

        <label for="exampleInputEmail1" class="superClass">Телефон</label>
        <input type="number" id="exampleInputEmail1" aria-describedby="emailHelp" v-model="append_driver_data[2]" placeholder="Введіть номер телефону" ref="pib" class="form-control">

        <label for="exampleFormControlTextarea1" class="superClass">Інформація</label>
        <textarea type="text" v-model="append_driver_data[3]" class="form-control" id="exampleFormControlTextarea1" rows="3" placeholder="Введіть інформацію про водія"></textarea>

        <input v-on:click="send_data" type="submit" value="Ok" class="button-1">
      </form>
    </div>
    <div v-else-if="status == 'append_driver_mark'" >
      <select v-model="select_driver"  class="tflir"> 
        <option value="" disabled selected >Обрати водія</option>
        <option v-for="item in drivers_arr" :key="item" v-bind:value="item" >
          {{item}}
        </option>
        
        
      </select>
      <input type="range" min="0" max="10" id="customRange2" v-model="you_mark" class="tflir">
        <h1 align="center">{{you_mark}}</h1>

      <label for="exampleFormControlTextarea1" class="superClass">Відгук</label>
      <textarea ref="you_resp" v-model="you_text" class="form-control" id="exampleFormControlTextarea2" rows="3" placeholder="Введіть свій відгук" ></textarea>
      <input  v-on:click="send_response" type="submit" value="Ok" class="button-1">
    </div>
    <div v-else-if="status == 'list_drivers'">
      <ol v-if="list_drivers" class="rectangle">
        <li v-for="item in list_drivers" :key="item" >
          <a href="#">{{item.star.toFixed(2)}} star, {{item.pib}}, phone - {{item.phone}}, {{item.email}}, {{item.info}}</a>
        </li>
      </ol>
      
    </div>
    <div v-else-if="status == 'find_driver'">
       <select v-model="select_driver"  class="tflir" > 
        <option value="" disabled selected>Обрати водія</option>
        <option v-for="item in drivers_arr" :key="item" v-bind:value="item">
          {{item}}
        </option>
      </select>
      <button v-on:click="get_driver_info"  type="button" class="button-2">Інформація про водія</button>
      <p></p>
      <div v-if="one_driver != null">
        {{one_driver.pib}}, {{one_driver.email}}, phone - {{one_driver.phone}}, {{one_driver.info}}
        <ul>
          <li v-for="item in one_driver.marks.length" :key="item">
            {{one_driver.dates[item-1]}}, {{one_driver.marks[item-1]}} star:: {{one_driver.resps[item-1]}}
          </li>
        </ul>
      </div>
    </div>
    <div v-else>
      error!
    </div>
  </div>

</template>
<!--

  status = main, append_driver, append_driver_mark, list_drivers, find_driver
  
 -->
<script>
import axios from 'axios'
function compare( a, b ) {
  if ( a.star > b.star ){
    return -1;
  }
  if ( a.star < b.star ){
    return 1;
  }
  return 0;
}
export default {
  name: 'app',
  data () {
    return {
      server: 'http://localhost:3000/',
      status: "main",
      append_driver_data: Array(4), // дані відправки форми
      drivers_arr: Array(),  // список водіїв для вибору зі списку)
      list_drivers: Array(), // рейтинг
      select_driver: '', // вибраний водій
      you_mark: 5, // оцінка водія
      you_text: '', // ваш відгук
      one_driver: null
    }
  },
  methods:{
    to_main(){
      this.status = "main";
      this.select_driver = '';
      this.you_text = '';
      this.you_mark = 5;
      this.drivers_arr = new Array();
    },
    to_find_driver(){
      axios.get(this.server+'drivers').then((response) => { 
        for(let i=0;i<response.data.length;i++){
          this.drivers_arr.push(response.data[i]); 
        }
      });
      this.status = "find_driver";
    },
    get_driver_info(){
      axios.post(this.server+'find_one_drivers', {
        pib: this.select_driver
      }).then((response) => { 
        this.one_driver = response.data;
      });
    },
    to_append_driver(){
      this.status = "append_driver";
    },
    to_append_driver_mark(){
      axios.get(this.server+'drivers').then((response) => { 
        for(let i=0;i<response.data.length;i++){
          this.drivers_arr.push(response.data[i]); 
        }
      });
      this.status = "append_driver_mark";
    },
    to_list_drivers(){
      axios.get(this.server+'drivers_list').then((response) => { 
        this.list_drivers =  response.data;
        this.list_drivers.sort(compare);
      });
      this.status = "list_drivers";
    },
    send_data(){
      if(this.append_driver_data[0] != null && this.append_driver_data[1] != null && this.append_driver_data[2] != null && this.append_driver_data[3] != null){
        axios.post(this.server+'drivers', {
          pib: this.append_driver_data[0],
          email: this.append_driver_data[1],
          phone: this.append_driver_data[2],
          info: this.append_driver_data[3]
        }).then((response) => {
            console.log("OK");
          }, (error) => {
            console.log(error);
        });  
        this.append_driver_data = new Array(4);
      }
      else{
        this.$refs.pib.focus();
      }
    },
    send_response(){
      if(this.select_driver != '' &&  this.you_text != '' ){
        axios.post(this.server+'drivers_resp', {
          driver: this.select_driver,
          mark: this.you_mark,
          text: this.you_text
        }).then((response) => {
            console.log("OK");
          }, (error) => {
            console.log(error);
        });  
        this.you_text = '';
      }
      else{
        this.$refs.you_resp.focus();
      }
    }
  }
}
</script>

<style>
body{
  background-image: url(bg.png) ;
}
.form-control{
  width: 300px;
  margin: auto;
}
.superClass{
  margin-left: 615px;
}
.center {
    margin: 40px auto;
}

.tflir {
    display: block;
    margin: 40px auto;
    background-color: #d1dde7;
    color: rgb(0, 0, 0);
    height: 40px;
}
.button-1 {
    color: #232323;
    background: transparent;
    border: 1px solid #232323;
    position: relative;
    font-size: 14px;
    letter-spacing: .3em;
    font-family: 'Montserrat', sans-serif;
    padding: 17px 34px 17px 39px;
    transition: .2s ease-in-out;
    cursor: pointer;
    display: block;
    margin: 40px auto;
    
}
.button-2 {
    color: #232323;
    background: transparent;
    border: 1px solid #232323;
    position: relative;
    font-size: 14px;
    letter-spacing: .3em;
    font-family: 'Montserrat', sans-serif;
    padding: 17px 34px 17px 39px;
    transition: .2s ease-in-out;
    cursor: pointer;
    display: block;
    margin: 20px auto;
}
.button-1:before {
    content: "";
    position: absolute;
    top: 5px;
    left: 5px;
    width: 100%;
    height: 100%;
    background: #ff9400;
    z-index: -1;
    transition: .25s ease;
    transform: translate(0, 0);
}
.button-2:before {
    content: "";
    position: absolute;
    top: 5px;
    left: 5px;
    width: 100%;
    height: 100%;
    background: #8399ae;
    z-index: -1;
    transition: .25s ease;
    transform: translate(0, 0);
}
.button-1:hover:before {
    transform: translate(4px, 4px);
}
.button-2:hover:before {
    transform: translate(4px, 4px);
}
.rectangle {
counter-reset: li; 
list-style: none; 
font: 14px "Trebuchet MS", "Lucida Sans";
padding: 0;
text-shadow: 0 1px 0 rgba(255,255,255,.5);
width: 1000px;
margin: auto;
}
.rectangle a {
position: relative;
display: block;
padding: .4em .4em .4em .8em;
margin: .5em 0 .5em 2.5em;
background: #D3D4DA;
color: #444;
text-decoration: none;
transition: all .3s ease-out;
}
.rectangle a:hover {background: #DCDDE1;}       
.rectangle a:before {
content: counter(li);
counter-increment: li;
position: absolute;
left: -2.5em;
top: 50%;
margin-top: -1em;
background: #9097A2;
height: 2em;
width: 2em;
line-height: 2em;
text-align: center;
font-weight: bold;
}
.rectangle a:after {
position: absolute;
content: "";
border: .5em solid transparent;
left: -1em;
top: 50%;
margin-top: -.5em;
transition: all .3s ease-out;
}
.rectangle a:hover:after {
left: -.5em;
border-left-color: #9097A2;
}

</style>
