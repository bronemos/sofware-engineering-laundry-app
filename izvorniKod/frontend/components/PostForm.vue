<template>
  <div class="form-box">
    <b-form id="form">
      <div class="form-group">
        <textarea
          class="form-control"
          rows="5"
          v-model="newPost.text"
          v-bind:placeholder="$t('writeText')"
        ></textarea>
        <div v-if="!imageUploaded" class="image-upload">
          <label for="file">
            <p>
              <img class="upload_icon" src="~@/static/images/upload.png" />
              {{ $t("attachPhoto") }}
              <input
                id="file"
                name="image"
                type="file"
                accept="image/*"
                ref="file"
                v-on:change="handleFileUpload()"
                hidden
              />
            </p>
          </label>
        </div>

        <div v-if="imageUploaded" class="small-img-preview">
          <img :src="imgUrl" />
          <button class="button-delete-photo" @click.prevent="refreshImage()">x</button>
        </div>
        <button
          class="submit-btn"
          @click.prevent="postForm"
          type="submit"
          :disabled="disableSubmit"
        >
          Post
        </button>
      </div>
    </b-form>
  </div>
</template>

<script>
export default {
  name: "PostForm",
  props: {
    type: String,
  },
  data() {
    return {
      newPost: {
        text: "",
        photo: null,
        posted_by: this.$auth.user.id,
      },
      imageUploaded: false,
    };
  },
  methods: {
    handleFileUpload() {
      this.newPost.photo = this.$refs.file.files[0];
      this.imageUploaded = true;
    },
    refreshImage(){
      // window.location.reload();
      this.imageUploaded = false;
    },
    async postForm() {
      try {
        let formData = new FormData();
        formData.append("text", this.newPost.text);
        formData.append("posted_by", this.newPost.posted_by);
        formData.append("type", this.type);
        if (this.newPost.photo) formData.append("photo", this.newPost.photo);
        let response = await this.$axios.post(`post/`, formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        });

        if (response.status == 201) {
          this.$toast.show("Post uspješno objavljen!", { duration: 8000 });
          let createdPost = response.data;
          document.getElementById("form").reset();
          this.imageUploaded = false;
          this.newPost.text = "";
          this.newPost.photo = null;
          this.$emit("post", createdPost);
        }
      } catch (e) {
        console.log(e);
      }
    },
  },
  computed: {
    disableSubmit() {
      return this.newPost.text == "";
    },

    imgUrl() {
      return URL.createObjectURL(this.newPost.photo);
    },
  },
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  font-family: sans-serif;
}
  button:focus {
    outline: none !important;
  }

.form-box {
  height: 300px;
  width: 1000px;
  position: relative;
  margin: 2% auto;
  background: white;
  padding: 5px;
  overflow: hidden;
  border-radius: 0.5rem;
  border: 2px solid white;
}
.submit-btn {
  width: 20%;
  padding: 10px 30px;
  cursor: pointer;
  display: block;
  margin: 20px;
  background: linear-gradient(to right, #4e43e2, #4fdee6);
  border: 0;
  outline: none;
  border-radius: 30px;
  color: white;
}
label {
  background-color: white;
  color: grey;
  padding-top: 2px;
  padding-bottom: 0px;
  font-family: sans-serif;
  border-radius: 1px;
  cursor: pointer;
  margin: 5px 1px 1px 1px;
}
.upload_icon {
  width: 5%;
  height: auto;
}
.small-img-preview {
  margin-left: 15px;
  margin-bottom: 10px;
  text-align: left;
  height: 100px;
  width: auto;
}

.small-img-preview > img {
  height: 100%;
  width: auto;
}
.button-delete-photo {
  border: 2px solid;
  border-color: grey;
  color: gray;
  background-color: white;
  border-radius: 5px;
  padding: 0px 5px;
  text-align: center;
  font-size: 12px;
  margin: 4px 2px;
  cursor: pointer;

}

</style>
