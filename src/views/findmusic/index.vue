<template>
    <div>
        <div class="container">
            <section>
                <div class="table-top">
                    <h3>歌曲类型</h3> tags
                </div>
                <el-card>
                    <div class="tagsinfo_box">
                        <div class="tags_style" v-for="item in tags" :key="item.id">
                            <div class="tags_name" @click="getTagsName(item.name)">{{item.name}}</div>
                        </div>
                    </div>
                </el-card>
            </section>
            <section>
                <div class="table-top">
                    <h3>精品歌单</h3> 共{{total}}首
                </div>
                <el-row>
                    <el-col :span="4"  v-for="item in playlist" :key="item.index">
                        <div class="imgbox">
                            <el-image :src="item.coverImgUrl" @click="toPlaylistDetail(item.id)" lazy></el-image>
                            <div class="listname">{{item.name}}</div>
                            <div class="playcount">
                                <i class="el-icon-headset"></i> 
                                {{item.playCount}} 
                                <i class="el-icon-video-play"></i>
                            </div>
                        </div>
                    </el-col>
                </el-row>
            </section>     
        </div>
    </div>
</template>

<script>
export default ({
    data(){
        return{
            playlist:[],
            playlistid:[],
            total:0,
            tags:[],
            cat:'华语'
        }      
    },
    created(){
        this.getPlaylist()
        this.getHighqualityTags()
        this.setActiveBoard()
    },
    methods:{
        setActiveBoard(){
            this.$store.commit('setActiveBoardId','findmusic')
        },
        async getHighqualityTags(){
            const {data:res} = await this.$http.get('/playlist/highquality/tags')
            this.tags = res.tags
        },
        async getPlaylist(){
            const {data:res} = await this.$http.get('/top/playlist/highquality?limit=15&cat='+this.cat);
            this.total = res.total
            // 注意！！！！ 这里是请求回来的是 playlists ，多一个s 复数
            this.playlist = res.playlists
        },
        getTagsName(tagsname){
            this.cat = tagsname
            this.getPlaylist()
        },
        // 跳转页面;这里的id是歌单的id，然后传过去
        toPlaylistDetail(id){
            this.$router.push({
                path:"/album",
                query: {   
                    id: id
                } 
            })
        }
    }
})
</script>

<style lang="less" scoped>
.container{
    .table-top{
        padding:10px 0;
    }
    .tagsinfo_box{
        display: flex;
        flex-wrap: wrap ;
        .tags_style{
            margin-right: 10px;
            .tags_name:hover{
                color: #FAD04B;
                cursor:pointer;
            }
        }
    }
    h3{
        display: inline;
    }
    .el-row{
        .el-col{
            width: 200px;
            height: 230px;
            .imgbox{
                width: 180px;
                position: relative;
                .el-image{
                    width: 180px;
                    height: 180px;
                    cursor:pointer;
                }
                .listname{
                    overflow:hidden;
                    text-overflow:ellipsis;  /* 溢出用省略号表示 */
                    white-space:nowrap;  /* 始终保持在一行显示 */
                }
                .playcount{
                    display: inline-block;
                    background-color: lightcyan;
                    opacity: 0.8;
                    padding: 0 5px;
                    line-height: 32px;
                    vertical-align: middle;
                    position:absolute;
                    width: 170px;   // 180px -5 -5 = 170px
                    top:148px;  // 180-32px = 148px

                    i:nth-child(2){
                        position: absolute;
                        top: 8px;       // 自身大小16px
                        right: 8px;
                    }
                }
            }         
        }    
    }
}
</style>