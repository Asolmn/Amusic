<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          controls
          :src="url"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="icon" alt="" />
          </div>
          <span class="name">{{ mvInfo.artistName }}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{ mvInfo.name }}</h2>
          <span class="date">发布：{{ mvInfo.publishTime }}</span>
          <span class="number">播放：{{ mvInfo.playCount }}次</span>
          <p class="desc">
            {{ mvInfo.desc }}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap">
        <p class="title">精彩评论<span class="number">({{ hotComment.length }})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in hotComment" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{ item.user.nickname }}：</span>
                <span class="comment">{{ item.content }}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{ item.beReplied[0].user.nickname }}：</span>
                <span class="comment">{{ item.beReplied[0].content }}}</span>
              </div>
              <div class="date">{{ item.time*0.001 | moment }}</div>
            </div>
          </div>
        </div>
      </div>
      <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">最新评论<span class="number">({{ Comment.length }})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in Comment" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{ item.user.nickname }}：</span>
                <span class="comment">{{ item.content }}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{ item.beReplied[0].user.nickname }}：</span>
                <span class="comment">{{ item.beReplied[0].content }}</span>
              </div>
              <div class="date">{{ item.time*0.001 | moment }}</div>
            </div>
          </div>
        </div>
      </div>
      <!-- 分页器 -->
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="5"
        :limit="limit"
      >
      </el-pagination>
    </div>
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div class="item" v-for="(item,index) in simiMvs" :key="index" @click="toMV(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'mv',
  data() {
    return {
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      // mv地址
      url: '',
      // 相关MV
      simiMvs: [],
      // mv信息
      mvInfo: {},
      // 歌手信息
      icon: '',
      // 热门评论
      hotComment: [],
      // 最新评论
      Comment: []
    };
  },
  created(){
    // MV视频
    axios({
      url: 'https://autumnfish.cn/mv/url',
      method: 'get',
      params: {
        id: this.$route.query.m
      }
    }).then(res=>{
      // console.log(res)
      this.url = res.data.data.url
    })

    // 相关MV
    axios({
      url: 'https://autumnfish.cn/simi/mv',
      method: 'get',
      params: {
        mvid: this.$route.query.m
      }
    }).then(res=>{
      console.log(res)
      this.simiMvs = res.data.mvs
      // 处理时间显示
      for(let i=0;i<this.simiMvs.length;i++){
        let min = parseInt(this.simiMvs[i].duration/1000/60)
        let sec = parseInt(this.simiMvs[i].duration/1000%60)

        if(min<10){
          min = '0'+min
        }

        if(sec<10){
          sec = '0'+sec
        }
        this.simiMvs[i].duration = min+':'+sec
        // 处理播放数量
        if(this.simiMvs[i].playCount>100000){
          this.simiMvs[i].playCount = parseInt(this.simiMvs[i].playCount/10000)+'w'
        }
      }
    })

    // MV信息
    axios({
      url: 'https://autumnfish.cn/mv/detail',
      method: 'get',
      params: {
        mvid: this.$route.query.m
      }
    }).then(res=>{
      console.log(res)
      this.mvInfo = res.data.data
      // 歌手信息
      axios({
        url: 'https://autumnfish.cn/artists',
        method: 'get',
        params: {
          id: this.mvInfo.artists[0].id
        }
      }).then(res=>{
        console.log(res)
        this.icon = res.data.artist.picUrl
      })
    })

    // 评论
    axios({
      url: 'https://autumnfish.cn/comment/mv',
      method: 'get',
      params: {
        id: this.$route.query.m,
        offset: 0
      }
    }).then(res=>{
      console.log(res)
      this.hotComment = res.data.hotComments
      this.Comment = res.data.comments
    })

  },

  methods: {
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page = val
      // 评论
      axios({
        url: 'https://autumnfish.cn/comment/mv',
        method: 'get',
        params: {
          id: this.$route.query.m,
          limit: 20,
          offset: (this.page-1)*20
        }
      }).then(res=>{
        console.log(res)
        this.Comment = res.data.comments
        this.total = this.Comment.length
      })
    },
    // 路由到MV页面
    toMV(id){
      this.$router.push(`/mv?m=${id}`)
    // MV视频
    axios({
      url: 'https://autumnfish.cn/mv/url',
      method: 'get',
      params: {
        id: this.$route.query.m
      }
    }).then(res=>{
      // console.log(res)
      this.url = res.data.data.url
    })

    // 相关MV
    axios({
      url: 'https://autumnfish.cn/simi/mv',
      method: 'get',
      params: {
        mvid: this.$route.query.m
      }
    }).then(res=>{
      console.log(res)
      this.simiMvs = res.data.mvs
      // 处理时间显示
      for(let i=0;i<this.simiMvs.length;i++){
        let min = parseInt(this.simiMvs[i].duration/1000/60)
        let sec = parseInt(this.simiMvs[i].duration/1000%60)

        if(min<10){
          min = '0'+min
        }

        if(sec<10){
          sec = '0'+sec
        }
        this.simiMvs[i].duration = min+':'+sec
        // 处理播放数量
        if(this.simiMvs[i].playCount>100000){
          this.simiMvs[i].playCount = parseInt(this.simiMvs[i].playCount/10000)+'w'
        }
      }
    })

    // MV信息
    axios({
      url: 'https://autumnfish.cn/mv/detail',
      method: 'get',
      params: {
        mvid: this.$route.query.m
      }
    }).then(res=>{
      console.log(res)
      this.mvInfo = res.data.data
      // 歌手信息
      axios({
        url: 'https://autumnfish.cn/artists',
        method: 'get',
        params: {
          id: this.mvInfo.artists[0].id
        }
      }).then(res=>{
        console.log(res)
        this.icon = res.data.artist.picUrl
      })
    })

    // 评论
    axios({
      url: 'https://autumnfish.cn/comment/mv',
      method: 'get',
      params: {
        id: this.$route.query.m,
        offset: 0
      }
    }).then(res=>{
      console.log(res)
      this.hotComment = res.data.hotComments
      this.Comment = res.data.comments
    })

    }
  }
};
</script>

<style></style>
