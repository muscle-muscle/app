<template>
<v-app>
    <v-app-bar
      app
      color="primary"
      dark
      height="80px"
    >
    <div>
      <span class="appname">
        Memo App
      </span>
    </div>
      <v-spacer></v-spacer>
      <v-col cols="4">
        <v-text-field id="access_token" v-model="loginID" :disabled="success"></v-text-field>
      </v-col>
      <v-btn target="_blank" text id="login" @click.once="login" :disabled="check">
        LOGIN
      </v-btn>
    </v-app-bar>

    <!-- memo -->
    <div id="memo">
      <v-col cols="6">
        <v-text-field  label="title" :disabled="memoable" id="memo-title" v-model="title" ></v-text-field>
        <v-textarea label="memo"  :disabled="memoable" id="memo-content" v-model="content"></v-textarea>
      </v-col>
      <v-btn id="save-memo" @click="save" :disabled="memoable">SAVE</v-btn>
      <v-btn id="delBtn" @click="deleteMemo" :disabled="memoable">DELETE</v-btn>
    </div>

    <!-- accordionpanel -->
    <div id="accordion" v-show="show">
      <v-container>
        <v-row>
          <v-col cols="3">
            <v-expansion-panels accordion>
              <v-expansion-panel v-for="(item,i) in memos" :key="i">
                <v-expansion-panel-header :id="'category-'+(i+1)" @click="searchMemo(i+1)">{{item.name}}</v-expansion-panel-header>
                <v-expansion-panel-content :id="'category-'+i+'title'" v-for="(item2,j) in memo" :key="j" 
                   class="memotitle" >
                  <v-btn text @click="viewMemo(item2.id)">{{item2.title}}</v-btn>
                </v-expansion-panel-content>              
              </v-expansion-panel>
            </v-expansion-panels>
            <v-btn id="new-memo" @click="addNewMemo">NEW</v-btn>
          </v-col>
        </v-row>
      </v-container>
    </div>

    <v-main>
      <router-view/>
    </v-main>
</v-app>
</template>

<script lang="ts">
import { Component,Vue} from 'vue-property-decorator';
//import axios from 'axios';
//import memo from '@/components/Memo.vue'

@Component({
  components:{
    //memo
  }
})
export default class App extends Vue {
  private pass='xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'
  private loginID= ''
  private disabled=true
  private success=false
  private show=false
  private memos:any =''
  private memo:any =''
  private title=''
  private content=''
  private memoable=true
  private memoId=0
  private Id=0
  private async created(){
    this.pass=
    this.pass.split('')
    .map(c => {
      switch (c) {
        case 'x': return (Math.random() * 16 | 0).toString(16);
        case 'y': return ((Math.random() * 4 | 0) + 8).toString(16);
        default: return c;
      }
    })
    .join('')
    console.log(this.pass)
    this.loginID=this.pass
  }

  

  private get check():boolean{
      if(this.pass===this.loginID){
          return !this.disabled
      }
      return this.disabled
  }

  private login(){
    //this.memoable=!this.memoable
    this.disabled=!this.disabled
    this.success=!this.success
    this.show=!this.show
    this.getCategory()
    .then(res=>{
      this.memos=res
      console.log(res)
    })
  }

  private async getCategory() {
    const result = await fetch(`https://challenge-server.tracks.run/memoapp/category`,
    {headers: {'X-ACCESS-TOKEN': this.pass}});
    return await result.json();
  }

/*   private addNewMemo(){
    this.memo.push({"id":1,"title":"New Memo"})
  } */

/*   private deleteMemo(){
    console.log(this.memo)
    console.log(2)
  } */
  
  private searchMemo(id:number){
    this.Id=id
    this.getMemo(id)
    .then(res=>{
      this.memo=res
      console.log(res)
    })
    .catch(e=>{
      console.log(e)
    })
  }

  private async getMemo(id:number){
    const result =await fetch(`https://challenge-server.tracks.run/memoapp/memo?&category_id=`+id,
    {headers: {'X-ACCESS-TOKEN': this.pass}});
    return await result.json()
  }

  private viewMemo(id:number){
    this.memoId=id
    this.getMemoCont(id)
    .then(res=>{
      this.title=res.title
      this.content=res.content
      console.log(res)
    })
    .catch(e=>{
      console.log(e)
    })
    if(this.memoable===true){
      this.memoable=!this.memoable
    }
  }

  private  async getMemoCont(id:number){
    const result =await fetch(`https://challenge-server.tracks.run/memoapp/memo/`+id,
    {headers: {'X-ACCESS-TOKEN': this.pass}});
    return await result.json()
  }


    private save(){
    //console.log(this.memoId)
    console.log(this.memoId)
    console.log(this.title)
    console.log(this.content)
    fetch(`https://challenge-server.tracks.run/memoapp/memo/${this.memoId}`,
    {headers:{"X-ACCESS-TOKEN":this.pass,"Content-Type":"application/json"},
    method:"PUT",
    body:JSON.stringify({"category_id":this.Id,"title":this.title,"content":this.content})})
    .then(res=>{console.log(res)})
    .catch(e=>{console.log(e)})
    
  }

  private addNewMemo(){
    this.add()
    .then(res=>{this.memo.push(res)})
    .catch(e=>{console.log(e)})
  }

  private async add(){
    const result = fetch(`https://challenge-server.tracks.run/memoapp/memo?&category_id=`+this.Id,
    {headers:{"X-ACCESS-TOKEN":this.pass,"Content-Type":"application/json"},
    method:"POST",
    body:JSON.stringify({"category_id":33,"title":"New Memo","content":"DO"})})
    .then(async (res)=>{return await res.json()})
    return result
  }


  private deleteMemo(){
    this.del()
    .then(res=>{this.searchMemo(this.Id)})
    .then(()=>{
      this.title=""
      this.content=""
      this.memoable=!this.memoable
    })
  }
  private async del(){
    const result= await fetch(`https://challenge-server.tracks.run/memoapp/memo/${this.memoId}`,
    {headers:{"X-ACCESS-TOKEN":this.pass,"Content-Type":"application/json"},
    method:"DELETE"});
    return  result
  }

  

/*     private deleteMemo(){
    fetch(`https://challenge-server.tracks.run/memoapp/memo/${this.memoId}`,
    {headers:{"X-ACCESS-TOKEN":this.pass,"Content-Type":"application/json"},
    method:"DELETE"})
    .then(res=>{console.log(res)})
    .catch(e=>{console.log(e)})
  } */
}
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#accordion{
  position:relative;
  top:-190px;
  left:-40px;
}

v-expansion-panel-content{
  border-top:1px solid black;
}

#memo{
  z-index:10;
  position: relative;
  top:100px;
  left:300px;
}

#new-memo{
  position: relative;
  top:10px;
  left: -60px;
}

#delBtn{
  position: relative;
  right:100px;
  top:-20px;
}

#save-memo{
position:relative;
left:-390px;
top:-20px;
}

.memotitle{
  cursor: pointer;
}
</style>
