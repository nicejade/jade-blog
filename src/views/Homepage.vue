<template>
  <div class="wrapper homepage">
    <div class="panel-default" v-loading="isLoading">
      <div class="panel-body">
        <div class="main-container">
          <div class="entry-list">
            <el-card class="box-card">
              <div slot="header" class="clearfix">
                <el-breadcrumb separator="/">
                  <el-breadcrumb-item :to="{ path: '/' }">{{ $t('homePage') }}</el-breadcrumb-item>
                  <el-breadcrumb-item>{{ $t('homepage') }}</el-breadcrumb-item>
                </el-breadcrumb>
              </div>

              <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane name="BaseInfo" :label="isUserSelf ? $t('baseInfo') : $t('baseInfo')">
                </el-tab-pane>
                <el-tab-pane
                  name="MyPublish"
                  :label="isUserSelf ? $t('myPublish') : $t('hisPublish')"
                >
                </el-tab-pane>
                <el-tab-pane name="MyLikes" :label="isUserSelf ? $t('myLikes') : $t('hisLikes')">
                </el-tab-pane>
                <el-tab-pane
                  name="MyDislikes"
                  v-if="isUserSelf"
                  :label="isUserSelf ? $t('myDislikes') : $t('hisDislikes')"
                >
                </el-tab-pane>
              </el-tabs>
              <el-card class="box-card base-info" v-if="isShowBaseInfo">
                <div slot="header" class="clearfix">
                  <img class="avatar" :src="userAvatar" :alt="$t('niceLinksStr')" />
                  <div class="info">
                    <p class="username">{{ mUserInfo.username }}</p>
                    <p class="nickname" v-if="mUserInfo.profile.nickname">
                      {{ mUserInfo.profile.nickname }}
                    </p>
                    <span class="gray" v-html="nicerNumText"></span>
                  </div>
                </div>
                <div class="form-group">
                  <label class="control-label">{{ $t('personalWebsite') }}:</label>
                  <p class="text-padding gray" v-if="!mUserInfo.profile.website">
                    {{ $t('noFill') }}
                  </p>
                  <a v-else :href="mUserInfo.profile.website" target="_blank" rel="noopener">
                    {{ mUserInfo.profile.website }}
                  </a>
                </div>
                <div class="form-group">
                  <label class="control-label">{{ $t('profile') }}:</label>
                  <preview-md :value="mUserInfo.profile.description || $t('noFill')"> </preview-md>
                </div>
              </el-card>
              <links-list v-else :pdata="myLinksList" :is-abstract="true" :is-loading="isLoading">
              </links-list>
            </el-card>
          </div>
          <aside-list></aside-list>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import metaMixin from 'mixins/metaMixin.js'
import PreviewMd from 'components/markdown/PreviewMd.vue'

export default {
  name: 'HomePage',

  mixins: [metaMixin],

  components: {
    PreviewMd,
  },

  data() {
    const vm = this
    return {
      title: vm.$t('homepage'),
      isLoading: false,
      isShowBaseInfo: true,
      activeName: 'BaseInfo',
      myLinksList: [],
      mUserInfo: {
        profile: {},
      },
      nicerNumText: '',
      tabApiObj: {
        MyPublish: 'getMyPublish',
        MyLikes: 'getMyLikes',
        MyDislikes: 'getMyDislikes',
      },
      tabDataObj: {
        MyPublish: null,
        MyLikes: null,
        MyDislikes: null,
      },
    }
  },

  computed: {
    isUserSelf() {
      return this.userInfo && this.userInfo.username === this.$route.params.id
    },
    userAvatar() {
      if (this.mUserInfo) {
        let defaultAvatar = 'https://image.nicelinks.site/default-avatar.jpeg'
        let userAvatar = this.mUserInfo.profile && this.mUserInfo.profile.avatar
        return userAvatar ? `/api/avatar/${userAvatar}` : defaultAvatar
      }
    },
  },

  created() {
    this.getUserInfoByUsername()
  },

  watch: {
    $lang(val) {
      this.updateDetailInfo()
    },
  },

  methods: {
    getUserInfoByUsername() {
      this.isLoading = true
      let params = { username: this.$route.params.id }
      this.$apis
        .getUserInfo(params)
        .then((result) => {
          this.mUserInfo = result
          this.updateDetailInfo()
        })
        .catch((error) => {
          this.$message.error(`${error}`)
        })
        .finally(() => {
          this.isLoading = false
        })
    },

    updateDetailInfo() {
      let cTemp = this.$t('nicerNumText').replace('@X', this.mUserInfo.number || 1)
      let joinTime = this.mUserInfo.activeTime || this.mUserInfo.createdAt
      joinTime = new Date(joinTime).Format('yyyy-MM-dd hh:mm:ss')
      this.nicerNumText = cTemp.replace('@TIME', joinTime)
    },

    requestApiUpdateList(index) {
      if (this.tabDataObj[index]) {
        this.myLinksList = this.tabDataObj[index]
        return
      }

      let currentApi = this.tabApiObj[index]
      let params = {
        username: this.$route.params.id,
        userId: (this.userInfo && this.userInfo._id) || '',
      }

      this.isLoading = true
      this.$apis[currentApi](params)
        .then((result) => {
          this.myLinksList = this.tabDataObj[index] = result
          this.isLoading = false
        })
        .catch((error) => {
          this.$message.error(`${error}`)
          this.isLoading = true
        })
    },

    handleClick(item) {
      this.isShowBaseInfo = item.name === 'BaseInfo'
      if (item.name !== 'BaseInfo') {
        this.requestApiUpdateList(item.name)
      }
    },

    onLinkClick(item) {
      item.website ? (document.location.href = item.website) : ''
    },
  },

  locales: {
    en: {
      baseInfo: 'Base Information',
      myPublish: 'My Publish',
      myLikes: 'My Likes',
      myDislikes: 'My Dislikes',
      hisPublish: 'His Publish',
      hisLikes: 'His Likes',
      hisDislikes: 'His Dislikes',
      nicerNumText: `<a href='/'>NICE LINKS</a> Member No. @X, Join in @TIME`,
      noFill: 'Not yet filled',
    },
    zh: {
      baseInfo: '基本信息',
      myPublish: '我的发布',
      myLikes: '我的点赞',
      myDislikes: '我的狂踩',
      hisPublish: '他的发布',
      hisLikes: '他的点赞',
      hisDislikes: '他的狂踩',
      nicerNumText: `<a href='/'>倾城之链</a>第 @X 号成员，加入于 @TIME`,
      noFill: '暂未填写',
    },
  },
}
</script>

<style lang="scss">
@import './../assets/scss/variables.scss';
@import './../assets/scss/mixins.scss';

.homepage {
  .main-container {
    .links-list {
      .el-card {
        padding: 15px;
      }
    }
    .el-card__header {
      padding: 15px;
    }
    .el-tabs {
      padding: 0 15px;
    }
    .el-card__body {
      padding: 0;
    }
    .el-tabs__header {
      margin: 0;
    }
    .box-card {
      .form-group {
        @include flex-box-center(row, start, center);
        .control-label {
          text-align: center;
          min-width: 90px;
          padding: 0 15px 0 0;
        }
      }
    }
    .base-info {
      padding: 2rem;
      .el-card__header {
        padding: 0px;
      }
      .clearfix {
        @include flex-box-center(row, start, center);
      }
      .avatar {
        float: left;
        border-radius: 50%;
        height: 6rem;
        width: 6rem;
        box-shadow: 0 0 0 2px #fff;
        position: relative;
        margin: 0;
      }
      .info {
        @include flex-box-center(column, space-around, left);
        width: calc(100% - 7rem);
        height: 8rem;
        float: left;
        margin-left: 1rem;
        .username {
          font-size: $font-large;
          font-weight: 500;
        }
      }
      .text-padding {
        padding: 10px 0;
      }
    }
  }
}

@media (max-width: 768px) {
  #app .wrapper.homepage {
    .main-container {
      .el-card__body {
        padding: 0;
      }
      .base-info {
        padding: 1rem;
      }
    }
  }
}
</style>
