<template>
  <div>
    <div id="ideasWraper">
      <idea v-for="idea in ideas" :idea="idea" :showUserInfo="showUserInfo"></idea>
    </div>
    <div class="text-center" id="ideasLoading" v-show="ideasLoading">
      <i class="fa fa-spin fa-spinner"></i>
    </div>
  </div>
</template>

<script>
  import Idea from './Idea';
    export default {
      name: "IdeaList",
      components: {
        idea: Idea,
      },
      created: function () {
        this.ideas = [];
        this.getIdeas(1);
        window.addEventListener('scroll',this.handleScroll);
        this.$root.$on('newIdea', this.handleNewIdea);
      },
      destroyed: function() {
        window.removeEventListener('scroll',this.handleScroll);
      },
      props: {
        endpoint: {type: String, default: '/ideas'},
        showUserInfo: {type: Boolean, default: true}
      },
      data: function () {
        return {
          ideas: [],
          page: {},
          ideasLoading: false
        }
      },
      watch: {
        endpoint: function () {
          this.ideas = [];
          this.getIdeas();
        }
      },
      methods: {
          getIdeas: function (page) {
            const _page = page == undefined ? 1 : page;
            this.$http.get(this.endpoint+'?page='+_page)
              .then(function (res) {
                this.ideas = this.ideas.concat(res.body.data);
                this.page = {current: res.body.current_page, last: res.body.last_page};
                this.ideasLoading = false
              }).catch(function () {
                this.ideasLoading = false;
            })
          },
          handleScroll: function () {
            if(document.body.scrollHeight - window.innerHeight - Math.round(window.pageYOffset) == 0) {
              if(this.page.current < this.page.last) {
                this.getIdeas(this.page.current + 1);
              }
            }
          },
          handleNewIdea: function (idea) {
            if(!this.$route.params.user_id || this.$route.params.user_id == idea.user._id) {
              this.ideas.unshift(idea);
            }
          }
      }
    }
</script>

<style scoped>
  #ideasWraper {
     margin: 0 -20px;
   }

  #ideasLoading {
    padding: 40px;
  }

  #ideasLoading i {
    font-size: 40px;
  }
</style>
