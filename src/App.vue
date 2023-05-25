<script setup>
import axios from 'axios';
axios.interceptors.request.use(request => {
  console.log('Request:', request);
  return request;
});

</script>
<script>
let location = "https://localhost:7090/api/";
let urlBase = null;
let url = null;
let ucetId = null;
let diteId = null;
let PohybId = null;
let getAll = false;
let postObject;
let putOp;
let putValue;
let filtry = {};

export default {
  data() {
    return {
      count: 0,
      radky: null,
      sloupce: null,
      putProperty: null,
      putTargetId: null
    }
  },
  methods: {
    UpdateData() {
      //let link = typeof comandoElegido == 'undefined' || comandoElegido == null ? url : url + comandoElegido;
      console.log('url:')
      console.log(url);
      
      this.radky = axios
        .get(url)
        .then(response => {
          if (typeof (response.data[0]) === 'undefined') {
            this.radky = [response.data];
            console.log(1);
          }
          else {
            this.radky = response.data;
            console.log(2);
          }
          this.sloupce = Object.keys(this.radky[0]);

          postObject = {};
          this.sloupce.forEach(sloupec => {
            if (sloupec != 'id' && sloupec != 'casVytoreni' && sloupec != 'zustatek') {
              //postObject[sloupec] = null;
            }

          });


        })
    },
    updateDataDelay() {
      setTimeout(this.UpdateData, 1000);
    },
    sendPatch(op, prop, value, url) {
      let path;
      path = prop == 'idMajitele' ? `/${prop}/-` : `/${prop}`;
      let putobject = [{
        "op": op,
        "path": path,
        "value": value
      }];
            console.log(op)
            axios.patch(url, putobject, {
        headers: {
          'Content-Type': 'application/json'
        }
      })

    
    },
    addFilters(dir){
      dir += '?';
      let first = true;
      Object.keys(filtry).forEach(f => {
        if(filtry[f].trim()!=''){
        if(first==false){dir+='&';}
        first=false;
        dir += f + '=' + filtry[f];}
        
      });
      console.log(dir);
      return dir;
    }
    
  }
};
</script>

<template>
  <input v-model="location" width="100%">
  <header>
    
    <div>
      <button
        @click="urlBase = location + 'deti/'; url = diteId == null ? urlBase : urlBase + diteId; getAll = diteId == null||'' ? true : false; UpdateData();">deti</button>
      <br>
      <input v-model="diteId" id="MiId">
    </div>
    <div>
      <button
        @click="urlBase = location + 'ucty/'; url = ucetId == null ? urlBase : urlBase + ucetId; getAll = ucetId == null||'' ? true : false; UpdateData();">ucty</button>
      <br>
      <input v-model="ucetId" id="MiId">
    </div>
    <div>
      <button
        @click="urlBase = location + 'ucty/' + ucetId + '/pohyby/'; url = PohybId == null ? urlBase : urlBase + PohybId; getAll = PohybId == null||'' ? true : false; UpdateData();">pohyby</button>
      <br>
      <input v-model="PohybId" id="MiId">
    </div>
  </header>
 
  <div>
    <table border="1">
      <thead>
        <tr>
          <th v-for="sloupec in sloupce">
            {{ sloupec }}
          </th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="radka in radky">
          <td v-for="sloupec in sloupce">
            {{ radka[sloupec] }}
            <button v-if="sloupec != 'id' && sloupec != 'casVytoreni' && sloupec != 'zustatek' && sloupec != 'ucty' && getAll == false"
              @click="putProperty = sloupec, putTargetId = radka.id">Edit</button>
          </td>
          <td><button @click="axios.delete(url + radka.id); updateDataDelay()">smazat</button></td>
        </tr>
        <tr v-if="getAll == true">
          <td v-for="sloupec in sloupce">
            <template v-if="sloupec != 'id' && sloupec != 'casVytoreni' && sloupec != 'zustatek' && sloupec != 'ucty' && sloupec != 'ucetId'">
              <input v-model="postObject[sloupec]" :placeholder="sloupec">
            </template>
            <template v-else>
              <label>Daný v databazi</label>
            </template>
          </td>
          <td><button @click="axios.post(url, postObject); updateDataDelay();">nový</button> {{ getAll }}</td>

        </tr>
      </tbody>
    </table>
  </div>
  <div>
    <table v-if="putProperty != null" border="1">
      <thead>
        <td>
          Id
        </td>
        <td>
          Vlastnost??
        </td>
        <td>
          Akce
        </td>
        <td>
          nová hodnota
        </td>
      </thead>
      <tr>
        <td>
          {{ putTargetId }}
        </td>
        <td>
          {{ putProperty }}
        </td>
        <td>
          <div v-if="putProperty === 'idMajitele'">
            <select v-model="putOp">
              <option value="add" selected>add</option>
              <option value="remove">remove</option>
            </select>
          </div>
          <div v-else>
            <select v-model="putOp">
              <option value="replace" selected>replace</option>
            </select>
          </div>
        </td>
        <td>
          <input v-model="putValue">
        </td>
      </tr>
      <tr>
        <button @click="sendPatch(putOp, putProperty, putValue, url); updateDataDelay()">poslat</button>
      </tr>
    </table>
  </div>
  <br>
 <div v-if="urlBase == location + 'ucty/' + ucetId + '/pohyby/'">
    <div v-for="sloupec in sloupce" class="class1" >
      <label>{{ sloupec }}</label>
      <input v-model="filtry[sloupec]">
    </div>
    <button @click="url = addFilters(urlBase); UpdateData();" >hledat</button>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    justify-content: flex-start;
    /* place-items: flex-start;
    flex-wrap: wrap; */
  }

  .class1 {
    display: flex;
  }
}
</style>
