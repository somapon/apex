<template>
    <div v-if="show">
        <div class="panel">
            <div class="panel-heading">
                <span class="panel-title">Receive Order {{model.number}}</span>
                <div>
                    <router-link :to="`/receive_orders`" class="btn" title="Go Back">
                        <i class="fa fa-arrow-left"></i>
                    </router-link>
                    <div class="btn-group">
                        <a :href="`/docs/receive_orders/${model.id}`" target="_blank" class="btn" title="View PDF">
                            <i class="fa fa-file-pdf-o"></i>
                        </a>
                        <a :href="`/docs/receive_orders/${model.id}?mode=download`" target="_blank"
                            class="btn" title="Download PDF">
                            <i class="fa fa-download"></i>
                        </a>
                    </div>
                </div>
            </div>
            <div class="panel-body">
                <div class="document">
                    <div class="document-heading">
                        <div class="row">
                            <div class="col col-4">
                                <p><strong>Received From:</strong></p>
                                <router-link :to="`/vendors/${model.vendor_id}`">
                                    <span>{{model.vendor.person}}</span><br>
                                    <span>{{model.vendor.company}}</span><br>
                                    <pre>{{model.vendor.billing_address}}</pre>
                                </router-link>
                            </div>
                            <div class="col col-4">
                                &nbsp;
                            </div>
                            <div class="col col-4">
                                <table class="document-summary">
                                    <tbody>
                                        <tr>
                                            <td>Status</td>
                                            <td><status :id="model.status_id" /></td>
                                        </tr>
                                        <tr>
                                            <td>Date:</td>
                                            <td>{{model.date}}</td>
                                        </tr>
                                        <tr>
                                            <td>Number:</td>
                                            <td>{{model.number}}</td>
                                        </tr>
                                        <tr>
                                            <td>Date:</td>
                                            <td>{{model.date}}</td>
                                        </tr>
                                        <tr>
                                            <td>Purchase Order:</td>
                                            <td>
                                                <router-link :to="`/purchase_orders/${model.purchase_order_id}`">
                                                    {{model.purchase_order.number}}
                                                </router-link>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                    <div class="document-body">
                        <table class="document-table">
                            <thead>
                                <tr>
                                    <th>Item Code</th>
                                    <th>Item Description</th>
                                    <th class="align-center">Qty Received</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="item in model.items">
                                    <td class="width-2">
                                        {{item.product.item_code}}
                                    </td>
                                    <td class="width-4">
                                        <router-link :to="`/products/${item.product_id}`">
                                            <pre>{{item.product.description}}</pre>
                                        </router-link>
                                    </td>
                                    <td class="width-1 align-center">
                                        {{item.qty}}
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <div class="document-footer row">
                        <div class="col col-9">
                            <strong>Internal Note</strong>
                            <div class="document-terms">
                                <pre>{{model.note}}</pre>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script type="text/javascript">
    import Vue from 'vue'
    import { get, post, byMethod } from '../../lib/api'
    import Status from '../../components/status/ReceiveOrder.vue'
    import {Dropdown} from '../../components/dropdown'
    export default {
        components: { Status, Dropdown },
        data() {
            return {
                show: false,
                model: {
                    items: [],
                    currency: {},
                    vendor: {},
                    purchase_order: {}
                }
            }
        },
        beforeRouteEnter(to, from, next) {
            get(`/api/receive_orders/${to.params.id}`)
                .then(res => {
                    next(vm => vm.setData(res))
                })
                // catch 422
        },
        beforeRouteUpdate (to, from, next) {
            this.show = false
            get(`/api/receive_orders/${to.params.id}`)
                .then(res => {
                    this.setData(res)
                    next()
                })
                //catch 422
        },
        methods: {
            deleteModel() {
                this.$refs.more.close()
                const r = confirm("Are you sure!")
                if(r != true) { return }
                this.$bar.start()
                byMethod('delete', `/api/receive_orders/${this.model.id}`)
                    .then(({data}) => {
                        if(data.deleted) {
                            this.$router.push('/receive_orders')
                            this.$message.success(`You have successfully deleted receive order!`)
                        }
                        this.$bar.finish()
                    })
                    .catch((err) => {})
            },
            markAs(status) {
                this.$refs.more.close()
                this.$bar.start()
                post(`/api/receive_orders/${this.model.id}/mark`, {status: status})
                    .then(({data}) => {
                        if(data.saved) {
                            Vue.set(this.model, 'status_id', data.status_id)
                            Vue.set(this.model, 'is_editable', data.is_editable)
                            this.$message.success(`You have successfully marked receive order!`)
                        }
                        this.$bar.finish()
                    })

            },
            setData(res) {
                Vue.set(this.$data, 'model', res.data.data)
                this.$title.set(`Receive Order - ${this.model.number}`)
                this.$bar.finish()
                this.show = true
            }
        }
    }
</script>
