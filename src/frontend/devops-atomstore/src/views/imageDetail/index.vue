<template>
    <div class="biz-container image-detail-wrapper" v-bkloading="{ isLoading }">
        <div class="biz-side-bar">
            <side-bar :nav="sideMenuNav" :side-menu-list="sideMenuList"></side-bar>
        </div>
        <router-view style="width: 100%" v-if="!isLoading"></router-view>
    </div>
</template>

<script>
    import sideBar from '@/components/side-nav'
    import { mapGetters } from 'vuex'

    export default {
        components: {
            sideBar
        },
        data () {
            return {
                isLoading: true,
                sideMenuList: [
                    {
                        list: [
                            {
                                id: 'imageOverview',
                                selectId: ['imageOverview'],
                                name: this.$t('store.概览'),
                                icon: 'icon-overview',
                                showChildren: false
                            },
                            {
                                id: 'imageDetail',
                                selectId: ['imageDetail', 'imageEdit'],
                                name: this.$t('store.详情'),
                                icon: 'icon-txt',
                                showChildren: false
                            },
                            {
                                id: 'imageSettings',
                                selectId: ['imageSettings'],
                                name: this.$t('store.设置'),
                                icon: 'icon-cog',
                                isOpen: false,
                                showChildren: true,
                                children: [
                                    {
                                        id: 'imageMemberManage',
                                        selectId: ['imageMemberManage'],
                                        name: this.$t('store.成员管理'),
                                        icon: ''
                                    }
                                ]
                            }
                        ]
                    }
                ]
            }
        },

        computed: {
            ...mapGetters('store', {
                'currentImage': 'getCurrentImage'
            }),

            sideMenuNav () {
                return {
                    backUrl: 'workList',
                    backType: 'image',
                    icon: 'atom-story',
                    title: this.currentImage.imageName,
                    url: ''
                }
            }
        },

        created () {
            Promise.all([this.initImage(), this.getMemInfo()]).catch((err) => {
                this.$bkMessage({ message: err.message || err, theme: 'error' })
            }).finally(() => (this.isLoading = false))

            if (['imageVisibleRange', 'imageMemberManage'].includes(this.$route.name)) {
                this.sideMenuList[0].list[2].isOpen = true
            }
        },

        methods: {
            goBack () {
                this.$router.push({
                    name: 'workList',
                    params: {
                        type: 'image'
                    }
                })
            },

            initImage () {
                const code = this.$route.params.imageCode
                return this.$store.dispatch('store/requestImageDetailByCode', code).then((res) => {
                    this.$store.dispatch('store/updateCurrentImage', res)
                })
            },

            getMemInfo () {
                const code = this.$route.params.imageCode
                return this.$store.dispatch('store/requestGetMemInfo', code).then((res) => {
                    const userInfo = {
                        isProjectAdmin: res.type === 'ADMIN',
                        userName: res.userName
                    }
                    this.$store.dispatch('store/updateImageMemInfo', userInfo)
                })
            }
        }
    }
</script>

<style lang="scss">
    .image-detail-wrapper {
        min-width: 1200px;
        height: 100%;

        .bk-table {
            th:first-child,
            td:first-child {
                padding-left: 20px;
            }
        }
    }
    .sub-view-port {
        height: calc(100% - 60px);
        overflow: auto;
    }
    .disable {
        cursor: not-allowed !important;
        color: #dfe0e5 !important;
    }
</style>
