<template>
  <div class="home">
    <div class="loaderContainer" v-if="isLoading">
      <hollow-dots-spinner
          :animation-duration="1000"
          :dot-size="15"
          :dots-num="3"
          color="#ff1d5e"
          class="loader"
      />
    </div>
    <div class="EmailContainer">
      <div v-for="i in Email" :key="i._id" class="Items" v-on:click="getmail(i._id)">
        <b-icon icon="star"></b-icon>
        <h5>{{ i.Sender_Username }}</h5>
        <h5>{{ i.Title }}</h5>
        <h5>{{ i.Send_Date.split("T")[0].split("-").reverse().join("-") }}</h5>
      </div>
      <!--button v-on:click="addmail">AddEmail</button-->
      <infinite-loading @infinite="loadMail">
        <div slot="no-more">Loaded All Messages</div>
        <div slot="no-results">There are no Email</div>
      </infinite-loading>
    </div>
  </div>
</template>

<script>
import InfiniteLoading from 'vue-infinite-loading';
import {HollowDotsSpinner} from 'epic-spinners'

export default {
  name: "Home",
  components: {
    InfiniteLoading,
    HollowDotsSpinner
  },
  data() {
    return {
      busy: false,
      isLoading: false
    }
  },
  computed: {
    Email() {
      return this.$store.state.Email;
    }
  },
  methods: {
    loadMail($state) {
      fetch("http://localhost:3000/email", {
        method: "POST",
        body: JSON.stringify({
          username: this.$store.getters.getUser.Username,
          page: this.$store.state.count
        }),
        headers: {
          "content-type": "application/json"
        }
      }).then(response => response.json()
      ).then(result => {
        if (result.status == "OK") {
          for (let x of result.content) {
            this.$store.state.Email.push(x)
          }
          this.$store.state.count += 1;
          $state.loaded()
          if (result.content.length < 20) {
            $state.complete()
          }
        } else {
          console.log(result)
        }
      })
    },
    getmail(id) {
      this.isLoading = true;
      fetch("https://speedwagonmailback.herokuapp.com/email/find", {
        method: "POST",
        body: JSON.stringify({
          id: id
        }),
        headers: {
          "content-type": "application/json"
        }
      }).then(response => response.json()
      ).then(result => {
        console.log(result)
        localStorage.openedmail = JSON.stringify(result.content)
        this.isLoading = false;
        this.$router.push("/dashboard/view")
      }).finally(() => {
        this.isLoading = false
      })
    },
  },

}
</script>

<style scoped lang=scss>
.home {
  background-color: $brown-100;
  display: flex;
  flex-direction: column;
  position: relative;

  .loader {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .loaderContainer {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    background: rgba(black, 0.2);
  }

  .EmailContainer {
    height: calc(100vh - 75px);
    overflow-y: auto;
    overflow-x: hidden;

    .Items {
      display: grid;
      grid-template-columns: 50px 248px calc(100% - 448px) 150px;
      &:hover{
        background-color:rgba(brown,0.2)
      }
      * {
        margin: 10px 15px;
      }
    }
  }
}
</style>