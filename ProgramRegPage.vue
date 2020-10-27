<template>
    <v-container
            fluid
            class="program-page"
    >
        <v-layout
                v-if="programData"
                row
                wrap
        >
            <v-flex
                    lg7
                    md6
                    pa-4
                    class="program-descr"
            >
                <vv-article
                        v-if="isWaitinglist"
                        class="article"
                        slug="PROGRAM_INFO"
                        :program-id="programData.id"
                />

                <vv-article
                        v-else
                        class="article"
                        slug="WAITINGLIST_INFO"
                />

                <v-layout
                        v-if="isWaitinglist"
                        row
                        wrap
                >
                    <v-flex
                            md6
                            sm12
                            mb-2
                    >
                        <vv-base-button
                                v-if="role !== 'mentee'"
                                color="accent"
                                :outline="role !== 'mentor'"
                                @click="enrollProgram(&quot;mentor&quot;)"
                        >
                            {{ $t('mentoring.programReg.mentorBtn') }}
                        </vv-base-button>
                    </v-flex>
                    <v-flex
                            md6
                            sm12
                            mb-2
                    >
                        <vv-base-button
                                v-if="role !== 'mentor'"
                                color="accent"
                                @click="enrollProgram(&quot;mentee&quot;)"
                        >
                            {{ $t('mentoring.programReg.menteeBtn') }}
                        </vv-base-button>
                    </v-flex>
                </v-layout>
            </v-flex>
            <v-flex
                    class="main-image hidden-xs-and-down"
                    :style="{ background: `url('${ programData.detailPhoto ? programData.detailPhoto.large : ''}') top center no-repeat` }"
            />
        </v-layout>
    </v-container>
</template>
<script>
    import { mapGetters } from 'vuex';

    import VvBaseButton from '@/shared/components/BaseButton';
    import VvArticle from '@/shared/components/Article';

    export default {
        name: 'ProgramReg',
        components: {
            VvBaseButton,
            VvArticle,
        },
        data() {
            return {
                program: null,
                programId: this.$route.params.id,
            };
        },
        computed: {
            ...mapGetters('global/user', {
                role: 'getPrimaryRole',
            }),
            programData() {
                return this.$store.getters['mentoring/programs/getProgram'];
            },
            isWaitinglist() {
                return (this.programData.state && this.programData.state.toLowerCase() === 'waitinglist') || !this.programData.state;
            },
        },
        created() {
            this.$store.dispatch('mentoring/programs/fetchProgram', this.programId);
        },
        methods: {
            enrollProgram(role) {
                const data = {
                    id: this.programId,
                    role,
                };

                this.$store.dispatch('mentoring/enrollments/enrollProgram', data)
                    .then(() => {
                        const erollId = this.$store.getters['mentoring/enrollments/getCurrentEnrollmentId'];
                        this.$router.push({ name: 'defaultEnrollment', params: { id: erollId } });
                    })
                    .catch((error) => {
                        /* eslint-disable */
                        console.log('ProgramRegPage', error);
                    });
            },
        },

    };
</script>
<style lang="scss" scoped>
    @import "~@/styles/index";

    .program-page {
        padding: 0;
        height: 100%;

        .program-descr {
            max-width: 480px;
        }

        .main-image {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: flex-start;
            height: calc(100% - 80px);
            background-size: cover;
            width: calc(100% - 580px);
            max-width: 500px;
            position: absolute;
            right: 0px;

            @media (max-width: 1024px) {
                max-width: 400px;
            }

        }
    }

    .page {
        display: grid;
        grid-template-areas: "content  image" "content  image";
        grid-template-columns: 1fr 520px;
        height: calc(100vh - 80px);
    }

    .content {
        grid-area: content;
        display: flex;
        flex-direction: column;
        align-items: center;
        padding-top: 40px;

        &-wrap {
            max-width: 500px;
        }
    }

    .reg-image {
        grid-area: image;
        background-repeat: no-repeat;
        background-position: center;
        background-size: cover;
    }

    .heading {
        margin-bottom: 40px;
    }

    .article {
        margin-bottom: 50px;
    }

    .btn-mentor {
        display: inline-block;
        margin-right: 17px !important;
    }
</style>