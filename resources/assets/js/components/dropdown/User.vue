<template>
    <div :class="[show ? 'dropdown dropdown--open': 'dropdown']" ref="inside">
        <div class="toggle" @click="toggle">
            <span class="dropdown-user">
                <i class="fa fa-user-circle-o"></i>
            </span>
            <slot name="title"></slot>
            <span class="typeahead-toggler">
              <i class="fa fa-angle-up" v-if="show"></i>
              <i class="fa fa-angle-down" v-else></i>
            </span>
        </div>
        <transition name="fade" mode="out-in">
            <div class="dropdown-menu" v-if="show">
                <div class="dropdown-inner">
                    <ul class="dropdown-list">
                        <slot></slot>
                    </ul>
                </div>
            </div>
        </transition>
    </div>
</template>
<script type="text/javascript">
    export default {
        props: ['title'],
        data() {
            return {
                show: false
            }
        },
        beforeDestroy() {
            this.close()
        },
        methods: {
            toggle() {
                if(this.show) {
                    this.close()
                } else {
                    this.open()
                }
            },
            open() {
                this.show = true
                this.$nextTick(() => {
                    window.addEventListener('click', this.clickOutEvent)
                })
            },
            close() {
              window.removeEventListener('click', this.clickOutEvent)
              this.show = false
            },
            clickOutEvent(evt) {
              var $dd = this.$refs.inside
              if (evt.target !== $dd && !$dd.contains(evt.target)) {
                this.close()
              }
            }
        }
    }
</script>
