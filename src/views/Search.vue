<template>
    <div>
        <el-card shadow="never">
            <div slot="header" class="clearfix">
                <el-tabs v-model="activeName" @tab-click="handleClick">
                    <el-tab-pane label="最新主题" name="latest">
                        <article v-for="(item, index) in articleList" :key="index" class="media">
                            <div class="media-left">
                                <figure class="image is-48x48">
                                    <img :src="item.avatar" style="border-radius: 5px;">
                                </figure>
                            </div>
                            <div class="media-content">
                                <div class="">
                                    <p class="ellipsis is-ellipsis-1">
                                        <el-tooltip class="item" effect="dark" :content="item.title" placement="top">
                                            <router-link :to="{path:`post/${item.id}`}">
                                                <span class="is-size-6">{{ item.title }}</span>
                                            </router-link>
                                        </el-tooltip>
                                    </p>
                                </div>
                                <nav class="level has-text-grey is-mobile  is-size-7 mt-2">
                                    <div class="level-left">
                                        <div class="level-left">
                                            <router-link class="level-item" :to="{ path: `/member/${item.username}/home` }">
                                                {{ item.alias }}
                                            </router-link>

                                            <span class="mr-1">
                        发布于:{{ dayjs(item.createTime).format("YYYY/MM/DD") }}
                      </span>

                                            <span
                                                    v-for="(tag, index) in item.tags"
                                                    :key="index"
                                                    class="tag is-hidden-mobile is-success is-light mr-1"
                                            >
                        <router-link :to="{ name: 'tag', params: { name: tag.name } }">
                          {{ "#" + tag.name }}
                        </router-link>
                      </span>

                                            <span class="is-hidden-mobile">浏览:{{ item.view }}</span>
                                        </div>
                                    </div>
                                </nav>
                            </div>
                            <div class="media-right" />
                        </article>
                    </el-tab-pane>
                    <el-tab-pane label="热门主题" name="hot">
                        <article v-for="(item, index) in articleList" :key="index" class="media">
                            <div class="media-left">
                                <figure class="image is-48x48">
                                    <img :src="`https://cn.gravatar.com/avatar/${item.userId}?s=164&d=monsterid`" style="border-radius: 5px;">
                                </figure>
                            </div>
                            <div class="media-content">
                                <div class="">
                                    <p class="ellipsis is-ellipsis-1">
                                        <el-tooltip class="item" effect="dark" :content="item.title" placement="top">
                                            <router-link :to="{name:'post-detail',params:{id:item.id}}">
                                                <span class="is-size-6">{{ item.title }}</span>
                                            </router-link>
                                        </el-tooltip>
                                    </p>
                                </div>
                                <nav class="level has-text-grey is-mobile  is-size-7 mt-2">
                                    <div class="level-left">
                                        <div class="level-left">
                                            <router-link class="level-item" :to="{ path: `/member/${item.username}/home` }">
                                                {{ item.alias }}
                                            </router-link>

                                            <span class="mr-1">
                        发布于:{{ dayjs(item.createTime).format("YYYY/MM/DD") }}
                      </span>

                                            <span
                                                    v-for="(tag, index) in item.tags"
                                                    :key="index"
                                                    class="tag is-hidden-mobile is-success is-light mr-1"
                                            >
                        <router-link :to="{ name: 'tag', params: { name: tag.name } }">
                          {{ "#" + tag.name }}
                        </router-link>
                      </span>

                                            <span class="is-hidden-mobile">浏览:{{ item.view }}</span>
                                        </div>
                                    </div>
                                </nav>
                            </div>
                            <div class="media-right" />
                        </article>
                    </el-tab-pane>
                </el-tabs>
            </div>

            <!--分页-->
            <!-- <pagination
                     v-show="page.total > 0"
                     :total="page.total"
                     :page.sync="page.current"
                     :limit.sync="page.size"
                     @pagination="init"
                     @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
             />-->
            <el-pagination
                    v-show="page.total > 0"
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="page.current"
                    :page-sizes="[5,10,20]"
                    :page-size="page.size"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="page.total">
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
    import {scrollTo} from "../utils/scroll-to";
    export default {
        name: "Search"
    }
</script>

<style scoped>

</style>