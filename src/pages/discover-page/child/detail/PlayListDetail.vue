<template>
  <div class="play-list-detail" v-if="playListDetail">
    <div class="detail-top">
      <div class="top-box">
        <img v-lazy="playListDetail.coverImgUrl" class="top-bg" />
        <div class="top-playlist">
          <div class="playlist-flex">
            <div class="playlist-cover">
              <img v-lazy="playListDetail.coverImgUrl" />
            </div>

            <div class="playlist-info">
              <p class="info-name van-multi-ellipsis--l2">
                {{ playListDetail.name }}
              </p>

              <div class="info-nickname">
                <img v-lazy="playListDetail.creator.avatarUrl" />
                <p>{{ playListDetail.creator.nickname }}</p>
                <BaseIcon icon="arrow-white" class="info-arrow-icon" />
              </div>

              <div class="info-description">
                <p class="van-multi-ellipsis--l2">
                  {{ playListDetail.description }}
                </p>
                <BaseIcon icon="arrow-white" class="info-arrow-icon" />
              </div>
            </div>
          </div>
          <div class="playlist-more-action">
            <div @touchstart="setPlayListId(playListDetail.id)">
              <BaseIcon icon="comment" />
              <p>{{ playListDetail.commentCount }}</p>
            </div>
            <div>
              <BaseIcon icon="share" />
              <p>{{ playListDetail.shareCount }}</p>
            </div>
            <div>
              <BaseIcon icon="download" />
              <p>下载</p>
            </div>
            <div>
              <BaseIcon icon="select" />
              <p>多选</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div>
      <div class="bottom-song-count">
        <div class="bottom-song-count-left">
          <BaseIcon icon="play-black" class="play-icon" />
          <p>播放全部</p>
          <p>(共{{ playListDetail.trackCount }}首)</p>
        </div>
        <div class="bottom-song-count-right">
          <p>+ 收藏({{ playCount }})</p>
        </div>
      </div>
    </div>
    <div class="bottom-scroll" ref="playListDetailScroll">
      <BaseSong
        v-for="(item, index) in playList"
        :index="index"
        :maxbr="item.maxbr"
        :mv="item.mv"
        :key="index"
        :id="item.id"
        :fee="item.songFee"
        @more="touchMore(item)"
        @playSong="playSong(playList, index)"
      >
        <template v-slot:name>
          <span>{{ item.name }}</span>
        </template>
        <template v-slot:author>
          <span v-for="(ar, index) in item.ar" :key="ar.id">
            {{ item.ar.length - index === 1 ? ar.name : `${ar.name} / ` }}
          </span>
          <span>
            {{ ` - ${item.al.name}` }}
          </span>
        </template>
      </BaseSong>
    </div>
    <TheMoreButtonPopup
      :more-button-popup-show="moreButtonShow"
      :song-object="authorInfo"
      @close="moreButtonShow = false"
      @touchstart="setSingerIdOne(authorInfo.ar)"
    />
    <TheSelectSingerPopup
      :singer-list="authorInfo.ar"
      :select-singer-popup-show="selectSingerShow"
      @close="selectSingerShow = false"
      @touchstart="setSingerId"
    />
  </div>
</template>

<script>
import BaseSong from "@/components/BaseSong";
import TheMoreButtonPopup from "@/components/TheMoreButtonPopup";
import TheSelectSingerPopup from "@/components/TheSelectSingerPopup";
import { playCountFormat } from "@/utils/utils";
import { getPlayListDetailById, getSongDetailById } from "@/api/song";

export default {
  // eslint-disable-next-line vue/no-unused-components
  components: { TheSelectSingerPopup, TheMoreButtonPopup, BaseSong },
  name: "PlayListDetailInfo",
  data() {
    return {
      moreButtonShow: false,
      selectSingerShow: false,
      authorInfo: {},
      playListDetail: undefined,
      playList: [],
      isTop: false
    };
  },
  methods: {
    // BaseSong的更多操作按钮
    touchMore(item) {
      this.authorInfo = item;
      this.moreButtonShow = true;
    },
    // 当只有一位歌手的时候，直接跳转至歌手详情，不止一位的时候先选择要看哪个
    setSingerIdOne(item) {
      if (item.length > 1) {
        this.moreButtonShow = false;
        this.selectSingerShow = true;
      } else {
        this.$router.push({ path: `/singer/${item[0].id}` });
      }
    },
    setPlayListId(id) {
      this.$router.push({ path: `/playlist-comments/${id}` });
    },
    playSong(playList, index) {
      this.$store.commit("playList/setPlayList", {
        playList,
        nowPlayIndex: index
      });
    },
    setSingerId(id) {
      this.$router.push({ path: `/singer/${id}` });
    }
  },
  computed: {
    playCount() {
      return playCountFormat(this.playListDetail.subscribedCount);
    }
  },
  async created() {
    const { playlist } = await getPlayListDetailById(this.$route.params.id);
    this.playListDetail = playlist;
    let trackIds = playlist.trackIds.map(item => item.id);
    const { songs, privileges } = await getSongDetailById(trackIds.toString());
    this.playList = songs;
    // 把获得到的音质信息添加到 音乐信息里（获取最大音质）
    privileges.forEach((item, index) => {
      this.playList[index]["maxbr"] = item.maxbr;
      this.playList[index]["songFee"] = item.fee;
    });
  }
};
</script>

<style scoped lang="scss">
.play-list-detail {
  width: 100%;
  height: 100%;
  overflow-y: auto;

  .detail-top {
    height: 5rem;

    .top-box {
      height: 100%;
      position: relative;
      overflow: hidden;
      background: #747d8c;

      .top-bg {
        width: 100%;
        height: 100%;
        -webkit-filter: blur(50px);
        -moz-filter: blur(50px);
        -ms-filter: blur(50px);
        filter: blur(50px);
      }

      .top-playlist {
        width: 90%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 5%;
        @include flex-box(column, center, center);

        .playlist-flex {
          width: 100%;
          @include flex-box(row, flex-start, center);

          .playlist-cover {
            width: 2.5rem;
            height: 2.5rem;

            img {
              width: 100%;
              height: 100%;
              border-radius: $default-radius;
              display: block;
            }
          }

          .playlist-info {
            width: calc(100% - 2.7rem);
            height: 2.5rem;
            padding-left: 0.2rem;
            @include flex-box(column, space-between);

            .info-name {
              font-size: 0.32rem;
              color: #ffffff;
              font-weight: bold;
            }

            .info-nickname {
              @include flex-box(row, flex-start, center);

              p {
                color: $white-smoke;
                font-size: 0.24rem;
                padding-left: 0.1rem;
              }

              img {
                width: 0.5rem;
                height: 0.5rem;
                border-radius: 50%;
              }
            }
            .info-description {
              width: 100%;
              @include flex-box(row, flex-start, center);

              p {
                width: calc(100% - 0.5rem);
                white-space: pre-line;
                color: $divider;
                font-size: 0.22rem;
              }
            }
          }
        }

        .playlist-more-action {
          width: 100%;
          margin: 0.3rem 0;
          @include flex-box(row, space-around, center);
          font-size: 0.21rem;
          color: #ffffff;
          text-align: center;

          p {
            margin-top: 0.1rem;
            line-height: 0.21rem;
          }
        }
      }
    }
  }

  .bottom-song-count {
    padding: 0.2rem 0;
    width: 100%;
    height: 0.6rem;
    background: #ffffff;
    border-top-left-radius: 0.5rem;
    border-top-right-radius: 0.5rem;
    position: relative;
    top: -0.55rem;
    overflow: hidden;
    @include flex-box(row, space-between, center);

    .bottom-song-count-left {
      margin-left: 0.35rem;
      @include flex-box(row, flex-start, center);

      .play-icon {
        border: 1px solid $title;
        border-radius: 50%;
        padding: 0.1rem;
        width: 0.15rem;
        height: 0.15rem;
      }

      p:nth-child(2) {
        font-size: 0.28rem;
        margin-left: 0.28rem;
        margin-right: 0.1rem;
        font-weight: bold;
      }

      p:nth-child(3) {
        color: $content;
      }
    }

    .bottom-song-count-right {
      margin-right: 0.35rem;
      color: #ffffff;
      padding: 0.13rem 0.15rem;
      background: $red;
      border-radius: 0.3rem;
    }
  }

  .bottom-scroll {
    position: relative;
    top: -0.55rem;
    margin: 0 3% -0.55rem;
  }
}
</style>
