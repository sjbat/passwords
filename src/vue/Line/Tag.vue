<template>
    <div :class="className" @click="openAction($event)" :data-tag-id="tag.id" :data-tag-title="tag.label">
        <i class="fa fa-star favorite" :class="{ active: tag.favorite }" @click="favoriteAction($event)"></i>
        <div class="favicon fa fa-tag" :style="{color: this.tag.color}" :title="tag.label"></div>
        <div class="title" :title="tag.label"><span>{{ tag.label }}</span></div>
        <slot name="middle"/>
        <div class="more" @click="toggleMenu($event)">
            <i class="fa fa-ellipsis-h"></i>
            <div class="tagActionsMenu popovermenu bubble menu" :class="{ open: showMenu }">
                <slot name="menu">
                    <ul>
                        <slot name="menu-top"/>
                        <!-- <translate tag="li" @click="detailsAction($event)" icon="info">Details</translate> -->
                        <translate tag="li" @click="editAction()" icon="edit">Edit</translate>
                        <translate tag="li" @click="deleteAction()" icon="trash">Delete</translate>
                        <slot name="menu-bottom"/>
                    </ul>
                </slot>
            </div>
        </div>
        <div class="date" :title="dateTitle">{{ getDate }}</div>
    </div>
</template>

<script>
    import $ from "jquery";
    import Translate from '@vc/Translate';
    import TagManager from '@js/Manager/TagManager';
    import Localisation from "@js/Classes/Localisation";
    import SearchManager from "@js/Manager/SearchManager";

    export default {
        components: {
            Translate
        },

        props: {
            tag: {
                type: Object
            }
        },

        data() {
            return {
                showMenu: false
            };
        },

        computed: {
            getDate() {
                return Localisation.formatDate(this.tag.edited);
            },
            dateTitle() {
                return Localisation.translate('Last modified on {date}', {date:Localisation.formatDate(this.tag.edited, 'long')});
            },
            className() {
                let classNames = 'row tag';

                if(SearchManager.status.active) {
                    if(SearchManager.status.ids.indexOf(this.tag.id) !== -1) {
                        classNames += ' search-visible';
                    } else {
                        classNames += ' search-hidden';
                    }
                }

                return classNames;
            }
        },

        methods: {
            favoriteAction($event) {
                $event.stopPropagation();
                this.tag.favorite = !this.tag.favorite;
                TagManager.updateTag(this.tag)
                          .catch(() => { this.tag.favorite = !this.tag.favorite; });
            },
            toggleMenu($event) {
                this.showMenu = !this.showMenu;
                this.showMenu ? $(document).click(this.menuEvent):$(document).off('click', this.menuEvent);
            },
            menuEvent($e) {
                if($($e.target).closest('[data-tag-id=' + this.tag.id + '] .more').length !== 0) return;
                this.showMenu = false;
                $(document).off('click', this.menuEvent);
            },
            openAction($event) {
                if($($event.target).closest('.more').length !== 0) return;
                this.$router.push({name: 'Tags', params: {tag: this.tag.id}});
            },
            detailsAction($event, section = null) {
                this.$parent.detail = {
                    type   : 'tag',
                    element: this.tag
                };
            },
            deleteAction(skipConfirm = false) {
                TagManager.deleteTag(this.tag);
            },
            editAction() {
                TagManager.editTag(this.tag)
                          .then((t) => {this.tag = t;});
            }
        }
    };
</script>

<style lang="scss">

    #app-content {
        .item-list {
            .row.tag {
                .favicon {
                    text-align     : center;
                    font-size      : 2.25rem;
                    vertical-align : top;
                }
            }
        }
    }

</style>