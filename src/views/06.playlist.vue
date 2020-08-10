<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <p class="title">{{ playlist.name }}</p>
        <div class="author-wrap">
          <img class="avatar" :src="playlist.creator.avatarUrl" alt="" />
          <span class="name">{{ playlist.creator.nickname }}</span>
          <span class="time">{{ playlist.createTime }} 创建</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item,index) in playlist.tags" :key="index">{{ item }}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <span class="desc">{{ playlist.description }}</span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item,index) in tracks" :key="index" @dblclick="playMusic(item.id)">
              <td>{{ index+1 }}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.name }}</span>
                    <span class="iconfont icon-mv"></span>
                  </div>
                  <span></span>
                </div>
              </td>
              <td>{{ item.ar[0].name }}</td>
              <td>{{ item.al.name }}</td>
              <td>{{ item.publishTime }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="评论" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">精彩评论<span class="number">({{ hotCount }})</span></p>
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
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">2020-02-12 17:26:11</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{ total }})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in comments" :key="index">
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
                <div class="date">2020-02-12 17:26:11</div>
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
        >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'playlist',
  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      // 歌单详情数据
      playlist: {},
      // 歌曲列表
      tracks: [],
      // 热门评论
      hotComment: [],
      hotCount: 0,
      // 最新评论
      comments: [],

    };
  },
  created(){
    // 歌单详情
    axios({
      url: 'https://autumnfish.cn/playlist/detail',
      method: 'get',
      params: {
        id: this.$route.query.d
      }
    }).then(res=>{
      console.log(res)
      this.playlist = res.data.playlist
      this.tracks = res.data.playlist.tracks

      for(let i=0;i<this.tracks.length;i++){
        let tracksid = this.tracks[i].id
        axios({
          url: 'https://autumnfish.cn/search',
          method: 'get',
          params: {
            keywords: tracksid
            }
          }).then(res=>{
            console.log(res)
            let min = parseInt(res.data.result.songs[0].duration/1000/60)
            let sec = parseInt(res.data.result.songs[0].duration/1000%60)
            if(min<10){
              min = '0'+min
            }  
            if(sec<10){
              sec = '0'+sec
            }

            this.tracks[i].publishTime = min+':'+sec
          })
      }
    })

    // 热门评论
    axios({
      url: 'https://autumnfish.cn/comment/hot',
      method: 'get',
      params: {
        id: this.$route.query.d,
        type: 2,
        limit: 10
      }
    }).then(res=>{
      // console.log(res)
      this.hotComment = res.data.hotComments
      this.hotCount = res.data.total
    })

    // 最新评论
    axios({
      url: 'https://autumnfish.cn/comment/playlist',
      method: 'get',
      params: {
        id: this.$route.query.d,
        limit: 10,
        offset: 0
      }
    }).then(res=>{
      // console.log(res)
      // 总个数
      this.total = res.data.total
      // 评论数据
      this.comments = res.data.comments
    })
  },
  methods: {
    // 播放歌曲
    playMusic(id){
      // console.log(id)
      axios({
        url: 'https://autumnfish.cn/song/url',
        method: 'get',
        params: {
          id
        }
      }).then(res=>{
        console.log(res)
        let url = res.data.data[0].url
        // 传递播放的url给父组件
        this.$parent.musicUrl = url
        console.log(this.$parent.musicUrl)
      })
      },
    // 评论
    getComment(){
    // 最新评论
      axios({
        url: 'https://autumnfish.cn/comment/playlist',
        method: 'get',
        params: {
          id: this.$route.query.d,
          limit: 10,
          offset: (this.page-1)*10,
        }
      }).then(res=>{
        // 总个数
        this.total = res.data.total
        // 评论数据
        this.comments = res.data.comments
      })
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page = val
      this.getComment()
    }
  }
};
</script>

<style >

</style>
