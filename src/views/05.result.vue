<template>
  <div class="result-container">
    <div class="title-wrap">
      <h2 class="title">{{ this.$route.query.q }}</h2>
      <span class="sub-title">找到{{ count }}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item,index) in songList" :key="index"
            @dblclick="playMusic(item.id)"
            >
              <td>{{ index+1 }}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.name }}</span>
                    <span class="iconfont icon-mv" v-if="item.mvid!=0"></span>
                  </div>
                  <span v-if="item.alias.length!=0">{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists" >
        <div class="items">
          <div class="item" v-for="(item,index) in playLists" :key="index"
          @click="toPlaylist(item.id)"
          >
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{ item.name }}</p>
          </div>

        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item,index) in mv" :key="index" @click="toMV(item.id)">
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
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'result',
  data() {
    return {
      // 搜索页切换
      activeIndex: 'songs',
      // 查询歌曲
      songList: [],
      // 歌单
      playLists: [],
      // 搜索结果数量
      count: 0,
      // 保存mv
      mv: []
 
    };
  },

  watch:{
    activeIndex(){
      // 歌曲 songs
      // 歌单 lists
      // mv
      
      let type = 1
      let limit = 10
      switch(this.activeIndex){
        case 'songs':
          type = 1
          break
        case 'lists':
          type = 1000
          break
        case 'mv':
          type = 1004
          limit = 8
          break


        default:
          break
      }

      axios({
        url: ' https://autumnfish.cn/search',
        method: 'get',
        params: {
          keywords: this.$route.query.q,
          type: type,
          limit: limit
        }
      }).then(res=>{
        // console.log(res)
        if(type==1){
          // 歌曲逻辑
          this.songList = res.data.result.songs
          this.count = res.data.result.songCount

          // 时间处理
          for(let i=0;i<this.songList.length;i++){
            let min = parseInt(this.songList[i].duration/1000/60)
            let sec = parseInt(this.songList[i].duration/1000%60)
            if(min<10){
              min = '0'+min
            }  
            if(sec<10){
              sec = '0'+sec
            }
            this.songList[i].duration = min+':'+sec
          }
        }else if(type==1000){
          console.log(res)
          this.playLists = res.data.result.playlists
          this.count = res.data.result.playlistCount

          // 处理播放量
          for(let i=0;i<this.playLists.length;i++){
            if(this.playLists[i].playCount>100000)
            this.playLists[i].playCount = parseInt(this.playLists[i].playCount/10000)+'w'
          }
        }else if(type==1004){
          console.log(res)
          this.mv = res.data.result.mvs
          this.count= res.data.result.mvCount

          // 处理播放数据和时间
          for(let i=0;i<this.mv.length;i++){
            // 时间
            let min = parseInt(this.mv[i].duration/1000/60)
            let sec = parseInt(this.mv[i].duration/1000%60)
            if(min<10){
              min = '0'+min
            }  
            if(sec<10){
              sec = '0'+sec
            }

            this.mv[i].duration = min+':'+sec

            // 播放次数
            if(this.mv[i].playCount>10000){
              this.mv[i].playCount = parseInt(this.mv[i].playCount/10000)+'w'
            }
          }

        }
      })      

    }
  },
  created(){
    axios({
      url: ' https://autumnfish.cn/search',
      method: 'get',
      params: {
        keywords: this.$route.query.q,
        type: '1',
        limit: 10
      }
    }).then(res=>{
      console.log(res)
      this.songList = res.data.result.songs
      this.count = res.data.result.songCount

      // 时间处理
      for(let i=0;i<this.songList.length;i++){
        let min = parseInt(this.songList[i].duration/1000/60)
        let sec = parseInt(this.songList[i].duration/1000%60)
        if(min<10){
          min = '0'+min
        }  
        if(sec<10){
          sec = '0'+sec
        }
        this.songList[i].duration = min+':'+sec
      }
    })
  },
  methods:{
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
    // 路由到歌单页面
    toPlaylist(id){
      this.$router.push(`/playlist?d=${id}`)
    },
    // 路由到MV页面
    toMV(id){
      this.$router.push(`/mv?m=${id}`)
    }
  }
};
</script>

<style >

</style>
