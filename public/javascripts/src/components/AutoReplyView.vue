<template>
  <div>
  
    <nav class="navbar navbar-default">
      <div class="navbar-header">
        <a class="navbar-brand">自动回复</a>
      </div>
      <div class="nav nav-justified">
        <form class="navbar-form navbar-left" role="search">
          <div class="form-group">
            <input type="text" class="form-control" placeholder="查找" v-model="critiria">
          </div>
        </form>
        <p class="navbar-text navbar-left">
          该版本为内部测试版本，如需退出请点击手机微信中的退出网页版。点击
          <span class="glyphicon glyphicon-send"></span>
          对TA自动回复
        </p>
      </div>
    </nav>

    <div class="row">
      <member
        v-for="member in showMembers"
        :member="member"
        :index="$index">
      </member>
    </div>

  </div>
</template>

<script>
import api from '../service/api'
import Member from './Member.vue'

module.exports = {

  name: 'AutoReplyView',

  components: {
    Member
  },

  data() {
    return {
      pluginName: 'autoReply',
      members: {},
      showMembers: {},
      critiria: ''
    }
  },

  methods: {
    getMembers() {
      return api.usersList(this.pluginName).then(members => {
        this.showMembers = this.members = members
      })
    }
  },

  watch: {
    critiria () {
      this.showMembers = this.members.filter(member => {
        return member.nickname.indexOf(this.critiria) > -1
      })
    }
  },

  events: {
    'switch-member': function (index) {
      let member = this.showMembers.splice(index,1)[0]

      api.userSwitch(this.pluginName, member.username).then(() => {
        member.switch = !member.switch
      })

      this.showMembers.unshift(member)
    }
  },

  route: {
    data () {
      api.checkLogin().then(() => {
        this.getMembers()
      }).catch(err => {
        this.$dispatch('login-error', err)
        setTimeout( () => {
            this.$router.go('/login')
          }, 2000)
      })
    }
  },
}
</script>