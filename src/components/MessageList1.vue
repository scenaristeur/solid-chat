<template>
  <div class="message-list">
    <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="600">


      <b-list-group>
        <b-list-group-item v-for="d in data" :key="d.id">
          <div class="row">
            <div class="col-10">
              <img  width="30" height="30" :src="d.pic" rounded="circle" alt=" " />
              {{ d.maker }}
            </div>
            <div class="col">
              {{ d.created }}
            </div>


          </div>
          <div class="col">
            {{ d.content }}
          </div>
        </b-list-group-item>

      </b-list-group>
    </div>
    <SolidChatSend />

  </div>
</template>

<script>
import store from "@/store";
import { fetchDocument } from 'tripledoc';
import { sioc, dct, foaf } from 'rdf-namespaces'
import infiniteScroll from 'vue-infinite-scroll' // https://www.npmjs.com/package/vue-infinite-scroll
// see https://medium.com/@fayazara/easiest-way-to-implement-infinite-scroll-in-vue-js-54a0a6c1111
import SolidChatSend from '@/components/SolidChatSend'
//var count = 0;
let solid = window.solid
console.log("SOLID",solid)

export default {
  store,
  solid,
  name: 'MessageList',
  components: {
    SolidChatSend
  },
  directives: {infiniteScroll},
  props: {
    msg: String
  },
  data: function () {
    return {
      data: [],
      busy: false,
      date: {},
      limite : {},
      root :"https://solidarity.inrupt.net/public/ChatTest",// "https://solidarity.inrupt.net/public/Solidarity",
      mainProps: {  }
      //  mainProps: { blank: true, blankColor: '#777', width: 75, height: 75, class: 'm1' }
    }
  },
  async created(){
    this.limite =  new Date("01/20/2020")
    this.today = new Date()
    this.date = this.today
    let withoutProtocol = this.root.split('//')[1]
    let sock = withoutProtocol.split('/')[0]+"/"
    this.socket = new WebSocket('wss://'+sock, ['solid.0.1.0']);
    console.log(this.socket)

    let p =  [this.root, this.date.getFullYear(), ("0" + (this.date.getMonth() + 1)).slice(-2), ("0" + this.date.getDate()).slice(-2), "chat.ttl"].join("/")
    this.$store.commit('chat/setFileUrl', p)

    this.socket.onopen = function() {
      console.log("this.socket opened")
      this.socket.send('sub '+p);

    }.bind(this)
    this.socket.onmessage = async function(msg) {
      if (msg.data && msg.data.slice(0, 3) === 'pub') {
        // resource updated, refetch resource
        console.log("updated",msg.data)
        //  this.history.push({type: "update", url: msg.data})
        //  this.$store.dispatch('chat/requestUpdate',msg.data.substring(4))
        await this.getMessages(msg.data.substring(4))
      }
    }.bind(this)
    await  this.getMessages(p)
  },
  methods: {
    async  getMessages(url){
    //  console.log(url)
      let messages = await  this.read(url)
      console.log("MESSAGES", messages)
      //  let data = this.data
      //let app = this
      /*  messages.reverse().forEach((m) => {
      app.data.indexOf(m) === -1 ? app.data.unshift(m) : console.log("This item already exists");
    });*/

    this.data = this.data.concat(messages)
   this.data = this.data.filter( this.onlyUnique )
    console.log(this.data)
  },

  loadMore: async function() {
    this.busy = true;

    //  setTimeout(async function()  {

    //  for (var i = 0, j = 10; i < j; i++) {
    if (this.limite <= this.date ){
      //  let date =  this.date
      //  console.log(this.date)
      let path = [this.root, this.date.getFullYear(), ("0" + (this.date.getMonth() + 1)).slice(-2), ("0" + this.date.getDate()).slice(-2), "chat.ttl"].join("/")
      //  console.log(path)

      let messages = this.read(path)
      this.data = this.data.concat(messages);

      //  this.data.push({ name: count++ , date:date});
      this.date.setDate(this.date.getDate() -1)

    }else{
      //console.log("over", this.limite)
      alert ("No message before "+this.limite)
    }


    //    }
    this.busy = false;
    //      }, 1000);
  },
  async    read(path){
    let messages = []
    try{
      const chatDoc = await fetchDocument(path);
      let  subjects = chatDoc.findSubjects();
      subjects = subjects.filter( this.onlyUnique )
      //  console.log(subjects)
      let triples = []

      for  (let s of subjects) {
        //    console.log("Compare",s.asRef(), this.root+"/index.ttl#this")
        if (s.asRef() != this.root+"/index.ttl#this"){
          //  console.log(s)
          //  let t = s.getTriples()
          let created = s.getString(dct.created)
          let content = s.getLiteral(sioc.content)
          let maker = s.getNodeRef(foaf.maker)

          /*if (this.$store.state.chat.users[maker] != undefined){
          pic = this.$store.state.chat.users[maker].pic
        }*//*else{
        let  p =  await solid.data[maker].vcard$hasPhoto || ""
        pic = `${p}`
        console.log(p)
        if (p != undefined && p != "undefined"){
        this.$store.commit('chat/setUser', {webId: maker, pic: pic})
      }else{
      pic = this.$store.commit('chat/setUser', {webId: maker, pic: ""})
    }
  }*/

  //this.$store.state.chat.users[maker].pic
  //  console.log("PIC",`${p}`)

  //  let parts = s.getAllNodeRefs(schema.hasPart)
  //  let parent = s.getNodeRef(schema.isPartOf)
  let t={id:s.asRef(),
    created: new Date(created).toLocaleString(),
    content: content,
    maker: maker,
    //  pic: `${p}`
    //  parts: parts,
    //  parent: parent
  }

  //  console.log(t)
  triples.push(t)
}

}


//console.log("USERS",this.$store.state.chat.users)

//  console.log(triples)
messages = triples.reverse()
}catch(e){
  //  console.log(e)

}
//  console.log("Messages",messages)
return messages

},
async getPic(webId){
  console.log(webId)
  let p = await solid.data[webId].vcard$hasPhoto

  let pic = `${p}`
  console.log(pic)
  return pic
},
onlyUnique(value, index, self) {
  return self.indexOf(value) === index;
},

}

}
</script>
