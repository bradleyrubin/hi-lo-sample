<template>
  <div class="hello">
  </div>
</template>

<script>

export default {
  components: {
  },
  props: {
    sampleProp: {
      type: String,
      required: false,
      default: '',
    },
  },
  data() {
    return {
      msg: '',
    };
  },
  created() {
  },
  beforeDestroy() {
  },
  computed: {
    sampleComputed() {
      return 1;
    },
  },
  methods: {
    sampleMethod() {
      console.log('Sample'); // eslint-disable-line no-console
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
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
