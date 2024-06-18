<template>
  <div id="app">
    print test<br />
    A4 document
    <ReportEditor />
  </div>
</template>

<script>
import ReportEditor from './components/ReportEditor.vue';

export default {
  name: 'App',
  components: {
    ReportEditor
  }
};
</script>

<style>
/* 必要に応じてスタイルを追加 */
</style>

