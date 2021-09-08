<template>
  <div id="app">
    <div class="title">H4Pay 관리 앱 배포</div>
    <div class="container">
      <b-field label="헌재 버전" horizontal> {{ currVersion }} </b-field>
      <b-field label="업데이트 대상 버전" horizontal>
        <b-input
          type="text"
          v-model="nextVersion"
          :placeholder="(currVersion + 0.001).toFixed(3)"
        />
      </b-field>
      <b-field label="변경사항" horizontal>
        <b-input type="textarea" v-model="changes" />
      </b-field>
      <b-field label="APK 파일 업로드" horizontal>
        <b-upload v-model="dropFile" drag-drop accept=".apk">
          <section class="section">
            <div class="content has-text-centered">
              <p>
                <b-icon icon="upload" size="is-large"> </b-icon>
              </p>
              <p v-if="typeof dropFile == 'function'">
                파일을 선택하거나 드래그하세요
              </p>
              <p v-if="typeof dropFile != 'function'">{{ dropFile.name }}</p>
            </div>
          </section>
        </b-upload>
      </b-field>
      <b-field>
        <b-button rounded style="float: right" @click="deploy">배포</b-button>
      </b-field>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      dropFile: File,
      currVersion: null,
      nextVersion: null,
      changes: "",
    };
  },
  created() {
    console.log(process.env);
    this.$axios
      .get(`${process.env.VUE_APP_API_URL}/version`)
      .then((res) => {
        this.currVersion = res.data.version;
      })
      .catch((err) => {
        alert(`현재 버전을 가져올 수 없습니다: ${err.message}`);
      });
  },
  methods: {
    deploy() {
      console.log(
        `${this.currVersion} | ${this.nextVersion} | ${this.changes}`
      );
      console.log(this.dropFile);
      let form = new FormData();
      form.append("nextVersion", this.nextVersion);
      form.append("changes", this.changes);
      form.append("apk", this.dropFile);

      this.$axios
        .post(`${process.env.VUE_APP_API_URL}/version/update`, form)
        .then((res) => {
          console.log(res.data);
        });
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  width: 100vw;
  padding: 10vw;
}
.container {
  border: 1px solid #ccc !important;
  border-radius: 15px;
  padding: 20px;
  padding-bottom: 6vh;
}

.upload {
  width: 100% !important;
}
.upload-draggable {
  width: 100% !important;
}
</style>
