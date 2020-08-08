<template>
  <div class="chat-scroller mb-4">
<!--
<section class="section">
    <ul>
          <li v-for="m in messages" :key="m.id">
            <div class="card">
              <header class="card-header">
                <div class="card-header-title">
                  <div class="col-10">
                    {{m.maker.split('/').slice(2,3)[0]}}
                  </div>
                  <div class="col">
                    {{m.created}}
                  </div>
                </div>
              </header>
              <div class="card-content">
                <div class="content">
                  <p>{{m.content}}</p>
                  <p>  {{m.id.split("#")[1]}}</p>
                </div>
              </div>
            </div>
          </li>
        </ul>
</section>-->
    <b-list-group>
      <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="limit">
      <b-list-group-item v-for="m of messages" :key="m.id" style="margin-bottom: 2rem;" data-aos="slide-up" data-aos-offset="100" data-aos-easing="ease-out-back">
        <div class="row card-header small">
          <div class="col">
            {{m.maker.split('/').slice(2,3)[0]}}
          </div>
        <div class="col-8">
        </div>

          <div class="col">
            {{m.created}}
          </div>
        </div>
        <div class="row">
          {{m.content}}
        </div>
        <div class="row">
          {{m.id.split("#")[1]}}
        </div>
      </b-list-group-item>
</div>
    </b-list-group>

<br><br>
  </div>
</template>

<script>
export default {
  name: 'ChatScroller',
  props: {
    messages: Array
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
