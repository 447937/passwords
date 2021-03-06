<template>
    <div class="item-details">
        <i class="fa fa-times" @click="closeDetails()"></i>
        <preview :image="object.preview" :icon="object.icon" :link="object.url" :host="object.website"/>
        <h3 class="title" :style="{'background-image': 'url(' + object.icon + ')'}">{{ object.label }}</h3>
        <div class="infos">
            <i class="fa fa-star favourite" :class="{ active: object.favourite }" @click="favouriteAction($event)"></i>
            <span class="date">{{ object.edited.toLocaleDateString() }}</span>
            <tags :password="object"/>
        </div>
        <tabs :tabs="getTabs">
            <pw-details slot="details" :password="object" />
            <notes slot="notes" :password="object"/>
            <div slot="share">
                <tabs :tabs="getSharingTabs">
                    <sharing slot="nextcloud" :password="object" class="password-share-nextcloud"/>
                    <qr-code slot="qrcode" :password="object"/>
                </tabs>
            </div>
            <revisions slot="revisions" :password="object"/>
        </tabs>
    </div>
</template>

<script>
    import Tabs from '@vc/Tabs';
    import Tags from '@vc/Tags';
    import API from '@js/Helper/api';
    import Translate from '@vc/Translate';
    import Events from '@js/Classes/Events';
    import Notes from '@vue/Details/Password/Notes';
    import QrCode from '@vue/Details/Password/QrCode';
    import Preview from '@vue/Details/Password/Preview';
    import PwDetails from '@vue/Details/Password/Details';
    import Revisions from '@vue/Details/Password/Revisions';
    import SettingsManager from '@js/Manager/SettingsManager';
    import PasswordManager from '@js/Manager/PasswordManager';
    import Sharing from '@vue/Details/Password/Sharing/Sharing';

    export default {
        components: {
            Tabs,
            Tags,
            Notes,
            QrCode,
            Sharing,
            Preview,
            PwDetails,
            Revisions,
            Translate
        },

        props: {
            password: {
                type: Object
            }
        },

        data() {
            return {
                object : this.password
            };
        },

        created() {
            Events.on('password.changed', this.refreshView);
        },

        beforeDestroy() {
            Events.off('password.changed', this.refreshView);
        },

        computed: {
            getTabs() {
                if(this.object.notes.length !== 0) {
                    return {details: 'Details', notes: 'Notes', share: 'Share', revisions: 'Revisions'};
                }
                return {details: 'Details', share: 'Share', revisions: 'Revisions'};
            },
            getSharingTabs() {
                if(SettingsManager.get('server.sharing.enabled') && (this.object.share === null || this.object.share.shareable === true)) {
                    return {nextcloud: 'Share', qrcode: 'QR Code'};
                }
                return {qrcode: 'QR Code'};
            }
        },

        methods: {
            favouriteAction($event) {
                $event.stopPropagation();
                PasswordManager.updatePassword(this.object)
                               .catch(() => { this.object.favourite = !this.object.favourite; });
            },
            closeDetails() {
                this.$parent.detail = {
                    type   : 'none',
                    element: null
                };
            },
            refreshView(event) {
                if(event.object.id === this.object.id) {
                    API.showPassword(this.object.id, 'model+folder+shares+tags+revisions')
                       .then((p) => {this.object = p;});
                }
            }
        },

        watch: {
            password(value) {
                this.object = value;
            }
        }
    };
</script>

<style lang="scss">
    .item-details {
        & > .fa.fa-times:nth-child(1) {
            position  : absolute;
            top       : 5px;
            right     : 5px;
            cursor    : pointer;
            padding   : 0.75rem;
            font-size : 1.3rem;
            color     : $color-black;
            z-index   : 1;

            &:hover {
                text-shadow : 0 0 2px $color-white;
            }
        }

        .title {
            white-space     : nowrap;
            text-overflow   : ellipsis;
            overflow        : hidden;
            font-size       : 1rem;
            font-weight     : 300;
            margin          : 0;
            background      : no-repeat 15px 15px;
            background-size : 32px;
            padding         : 15px 15px 2px 57px;
            line-height     : 32px;
        }

        .infos {
            padding : 0 15px 20px;
            color   : $color-grey-dark;

            .favourite {
                cursor : pointer;

                &:hover,
                &.active {
                    color : $color-yellow;
                }
            }

            .tags-container {
                position    : static;
                display     : inline;
                color       : $color-black-light;
                margin-left : 3px;
            }
        }

        > .tab-container {
            padding : 0 15px 15px;
        }

        @media (max-width : $mobile-width) {
            .title {
                margin-bottom : 1rem;
            }
        }
    }
</style>