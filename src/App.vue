<template>
  <div class="p-d-flex p-flex-column p-jc-center">
    <div class="p-mr-2" style="min-height: 90vh">
      <DORemark msg="Generate DO Remark" />
    </div>
    <div class="p-mt-2">
      <div class="p-text-center">
        <span> Pollux Dev &copy; {{ year }} </span>
      </div>
    </div>
  </div>
</template>

<script>
import DORemark from './components/DORemark.vue';

export default {
  name: 'App',
  components: {
    DORemark
  },
  computed: {
    year() {
      const date = new Date();
      const year = date.getFullYear();
      return year;
    }
  }
};
</script>

<style>
html,
body {
  padding: 0px;
  margin: 0px;
}
</style>
