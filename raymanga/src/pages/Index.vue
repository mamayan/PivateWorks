<template>
  <div class="index-wrapper">
    <div class="cover-wrapper">
      <div class="cnt">
        <div class="img-box">
          <img class="cover-img" :src="`//img1.raymangaapp.com${bookInfo.cover_url}`" alt="">
          <span></span>
        </div>
        <div class="share" @click="popShareMask">
          <span class="i-share"></span>
          <span>share</span>
        </div>
        <div class="cover-title">
          <div class="title">{{bookInfo.book_name}}</div>
          <div class="author">{{bookInfo.author}}</div>
        </div>
      </div>
      <div class="summary">
        <span class="title">-INFO-</span>
        <span :class="classList">{{bookInfo.summary}}</span>
        <span class="btn-more" @click="getMore">-{{btnTxt}}-</span>
      </div>

    </div>
    <chapterList :chapterArr="chapterArr"></chapterList>
    <BottomPanel :userLang="userLang"></BottomPanel>
    <shareMask :isMaskShow="isShow" @popShareMask="popShareMask"></shareMask>
  </div>
</template>

<script>
import lang from '../vendors/lang.js';
import util from '../vendors/util.js';
import BottomPanel from '../components/BottomPanel.vue';
import chapterList from '../components/ChapterList.vue';
import shareMask from '../components/shareMask.vue';

export default {
  name: 'index',
  data() {
    return {
      bookId: util.getQuery('bookid'),
      chapterArr: [],
      bookInfo: {},
      // btnTxt: this.userLang.btn,
      classList: ['content'],
      userLang: {},
      btnTxtTemp: '',
      // maxNum: 0
      isShow: false,
    };
  },
  computed: {
    btnTxt() {
      return this.btnTxtTemp;
    },
  },
  async mounted() {
    await this.getBookInfo();
    //获取用户设备语言 包含常规浏览器和ie浏览器
    let langKey = (navigator.language || navigator.userLanguage).slice(0, 2);
    //切换ui至相应语言
    this.userLang = lang[langKey];
    this.btnTxtTemp = this.userLang.btnOpen;

    //调用统计首页pv函数
    this.indexPvHandle()
  },
  methods: {
    /**  
     * 统计首页pv函数,id可从链接获取,ChapterStar.vue有获取cookie uuid工具函数
    */
    indexPvHandle(){
      console.log('index pv')
    },
    /**
     * 获取对应bookid书籍信息
     */
    getBookInfo() {
      const getBookInfo =
        '//previewapi.raymangaapp.com/previewapi/v1/common/getBookInfo';
      this.$axios
        .post(getBookInfo, {
          book_id: this.bookId,
        })
        .then(res => {
          /**
           * 状态码
           * 1     ：成功
           * 2000  ：常规错误
           * 2001  ：参数错误
           * 2002  ：数据库连接错误
           */
          const { code, bookInfo } = res.data;
          if (code === 1) {
            this.chapterArr = bookInfo.chapter_list;
            this.bookInfo = bookInfo.book_info;
            // 本书章节最大数
            // this.maxNum = this.chapterArr[this.chapterArr.length-1].chapter_number
          } else if (code === 2000) {
            this.$toast('常规错误(2000)');
          } else if (code === 2001) {
            this.$toast('参数错误(2001)');
          } else if (code === 2002) {
            this.$toast('数据库连接错误(2002)');
          }
        })
        .catch(error => {
          this.$toast('网络繁忙，请稍后再试');
        });
    },
    /**
     * 改变展开按钮文本及摘要内容高度
     */
    getMore() {
      this.classList =
        this.classList.indexOf('more') > -1 ? ['content'] : ['content', 'more'];
      this.btnTxtTemp =
        this.btnTxtTemp == this.userLang.btnOpen
          ? this.userLang.btnClose
          : this.userLang.btnOpen;
    },
    /**
     * 点击分享按钮弹出选择平台浮层
     */
    popShareMask() {
      this.isShow = !this.isShow;
    },
  },
  components: {
    BottomPanel,
    chapterList,
    shareMask,
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
@import '../assets/scss/common.scss';

.index-wrapper {
  width: 100%;
  padding-bottom: 60px;
  overflow-y: auto;
}
.cover-wrapper {
  position: relative;
  width: 100%;
  .cnt {
    position: relative;
  }
  .img-box {
    position: relative;
    z-index: 0;
    height: rem(720px);
    .cover-img {
      position: absolute;
      width: 100%;
      height: 100%;
      font-size: 0;
    }
    span {
      position: absolute;
      z-index: 3;
      bottom: -2px;
      display: inline-block;
      width: 100%;
      height: rem(180px);
      // background: #000;
      background: linear-gradient(to top, #000, rgba(0, 0, 0, 0));
    }
  }

  .cover-title {
    position: absolute;
    bottom: rem(20px);
    z-index: 1;
    width: 100%;
    // height: 100%;
    color: #fff;
    // padding-top: 188px;
    .title {
      font-size: rem(42px);
      padding: 0 rem(40px);
      box-sizing: border-box;
    }
    .author {
      color: #ffa800;
      font-size: rem(40px);
      font-weight: bold;
      padding: 0 rem(40px);
      box-sizing: border-box;
      &:before {
        content: '';
        position: relative;
        transform: translateY(15%);
        display: inline-block;
        width: rem(50px);
        height: rem(50px);
        background: url(#{$base}/ic_auther.png) center center no-repeat/100%;
        margin-right: 3px;
      }
    }
  }
  .summary {
    padding: 0 rem(40px);
    box-sizing: border-box;
    color: #fff;
    background: #000;
    display: flex;
    flex-direction: column;
    align-items: center;
    .title {
      font-size: rem(36px);
      font-weight: bold;
      @include ellipsis(2);
    }
    .content {
      font-size: rem(32px);
      @include ellipsis(3);
    }
    .content.more {
      -webkit-line-clamp: 9999999;
    }
    .btn-more {
      font-size: rem(28px);
    }
  }
  .share {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-left: auto;
    color: #fff;

    position: absolute;
    top: rem(30px);
    right: rem(40px);
    .i-share {
      display: inline-block;
      width: rem(64px);
      height: rem(64px);
      font-size: rem(24px);
      background: url(#{$base}/ic_reader_navigation_share_white.png) center
        center no-repeat/100%;
    }
  }
}
</style>
