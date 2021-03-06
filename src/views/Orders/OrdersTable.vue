<template>
    <div class="card shadow">
        <div  class="card-header d-flex justify-content-between">
            <h3>Orders</h3>
            <div>
                <span class="pr-3">Filter</span>
                <base-button size="sm" v-if="pageLoading"><i class="ni ni-settings-gear-65 spin"></i></base-button>
                <base-dropdown v-else position="right">
                    <base-button slot="title" :type="badgeClass(status)" class="dropdown-toggle" size="sm">
                        {{ status }}
                    </base-button>
                    <a :class="['dropdown-item', `text-black`]" @click="status = 'ALL'">ALL</a>
                    <a :class="['dropdown-item', `text-${badgeClass('PENDING')}`]" @click="status = 'PENDING'">PENDING</a>
                    <a :class="['dropdown-item', `text-grey`]" @click="status = 'PROCESSING'">PROCESSING</a>
                    <a :class="['dropdown-item', `text-${badgeClass('READY')}`]" @click="status = 'READY'">READY</a>
                    <a :class="['dropdown-item', `text-${badgeClass('OUTFORDELIVERY')}`]" @click="status = 'OUTFORDELIVERY'">OUTFORDELIVERY</a>
                    <a :class="['dropdown-item', `text-${badgeClass('DELIVERED')}`]" @click="status = 'DELIVERED'">DELIVERED</a>
                    <a :class="['dropdown-item', `text-${badgeClass('CANCELLED')}`]" @click="status = 'CANCELLED'">CANCELLED</a>
                </base-dropdown>
            </div>
        </div>
        <div class="card-body d-flex flex-row justify-content-between flex-wrap">
            <div
                v-for="(order, index) in tableData"
                :key="order.order_id"
                class="card border-light col-md-6 mb-2"
            >
                <div class="p-4">
                    <span :class="['badge', `badge-${badgeClass(order.order_status)}`]" >{{ order.order_status }}</span>
                    <div class="d-flex text-muted">
                        <h3 class="m-0 pr-2 text-muted">Order Date</h3>
                        <span>{{ formatDate(order.order_date) }}</span>
                    </div>
                    <div class="d-flex">
                        <h3 class="m-0 pr-2">Name</h3>
                        <span>{{order.user.user_profile.name || order.user.email}}</span>
                    </div>
                    <div class="d-flex flex-column">
                        <h3 class="m-0 pr-2">Address</h3>
                        <div class="d-flex flex-column">
                        <span>{{order.delivery_address.address1}}</span>
                        <span>{{order.delivery_address.address2}}</span>
                        <span>{{order.delivery_address.address3}}</span>
                        <span>{{order.delivery_address.city}}</span>
                        <span>{{order.delivery_address.distrcit}}</span>
                        <span>{{order.delivery_address.state}}</span>
                        <span>{{order.delivery_address.pincode}}</span>
                        <span><small class="font-weight-bold">Landmark:</small> {{order.delivery_address.landmark}}</span>
                        </div>
                    </div>
                    <div class="d-flex">
                        <h3 class="m-0 pr-2">Phone</h3>
                        <span>{{ order.delivery_address.phone || order.user.phone }}</span>
                    </div>
                </div>
                <div class="card-footer">
                    <base-button @click="$set(modals, index, true)" size="sm">View Order</base-button>
                    <base-button size="sm" v-if="loading[index]"><i class="ni ni-settings-gear-65 spin"></i></base-button>
                    <base-dropdown v-else>
                        <base-button slot="title" :type="badgeClass(order.order_status)" class="dropdown-toggle" size="sm">
                            {{ order.order_status }}
                        </base-button>
                        <a :class="['dropdown-item', `text-${badgeClass('PENDING')}`]" @click="changeStatus('PENDING', index)">PENDING</a>
                        <a :class="['dropdown-item', `text-grey`]" @click="changeStatus('PROCESSING', index)">PROCESSING</a>
                        <a :class="['dropdown-item', `text-${badgeClass('READY')}`]" @click="changeStatus('READY', index)">READY</a>
                        <a :class="['dropdown-item', `text-${badgeClass('OUTFORDELIVERY')}`]" @click="changeStatus('OUTFORDELIVERY', index)">OUTFORDELIVERY</a>
                        <a :class="['dropdown-item', `text-${badgeClass('DELIVERED')}`]" @click="changeStatus('DELIVERED', index)">DELIVERED</a>
                        <a :class="['dropdown-item', `text-${badgeClass('CANCELLED')}`]" @click="changeStatus('CANCELLED', index)">CANCELLED</a>
                    </base-dropdown>
                </div>
                <!-- Modal -->
                <modal :show.sync="modals[index]">
                    <template slot="header">
                        <h5 class="modal-title">Order Items</h5>
                    </template>
                    <div>
                        <div class="table-responsive">
                            <base-table
                                :data="order.items"
                            >
                                <template slot="columns">
                                    <th>Item Name</th>
                                    <th>Quantity</th>
                                    <th>Unit</th>
                                    <th class="text-right">Total Price</th>
                                </template>
                                <template slot-scope="{row}">
                                    <td>
                                        {{ row.item_details.item_name }}
                                    </td>
                                    <td>
                                        {{ parseFloat(row.quantity) }}
                                    </td>
                                    <td>
                                        {{ row.unit }}
                                    </td>
                                    <td class="text-right">
                                        {{ parseFloat(row.total_price) }}
                                    </td>
                                </template>
                            </base-table>
                        </div>
                        <div class="p-3 mt-4 font-weight-bold text-right">
                            <span class="mr-3">Grand Total:</span><span>{{ order.grand_total }}</span>
                        </div>
                    </div>
                    <template slot="footer">
                        <base-button type="secondary" @click="$set(modals, index, false)">Close</base-button>
                    </template>
                </modal>
                <!-- Modal -->
            </div>
        </div> <!-- Card body -->
        <div>
            <base-pagination 
                :page-count="total_pages"
                v-model="page"
                align="center">
            </base-pagination>
        </div>
    </div> <!-- Card -->
</template>
<script>
import io from 'socket.io-client';

export default {
    name: 'orders-table',
    props: {
        type: {
            type: String
        },
    },
    data() {
        return {
            tableData: [],
            count: 0,
            modals: [],
            loading: [], // for status change.
            pageLoading: false, // for page loading
            total_pages: 0,
            per_page: 10,
            page: 1,
            status: 'ALL'
        }
    },
    computed: {
        baseUrl() {
            // base url of api server where images are uploaded.
            return this.$store.getters.apiUrl;
        },
        storeId() {
            return this.$store.getters.getUser.store[0].store_id;
        }
    },
    watch: {
        // whenever page changes, call getOrders
        page() {
            this.getOrders(this.storeId, this.page, this.per_page, this.status);
        },
        per_page() {
            this.getOrders(this.storeId, this.page, this.per_page, this.status);
        },
        status() {
            this.getOrders(this.storeId, this.page, this.per_page, this.status);
        }
    },
    methods: {
        getOrders(store_id, page = 1, per_page = 10, status = 'ALL') {

            status = status === 'ALL' ? null : status;

            this.$axios({
                method: 'get',
                url: '/orders/view',
                params: {
                    store_id: store_id,
                    page: page,
                    per_page: per_page,
                    order_status: status
                },
            }).then((response) => {
                let data = response.data.data.orders;

                this.tableData = data.rows;
                this.count = data.count;
                this.total_pages = data.total_pages;
                // initialize array
                this.modals = Array.from({length: this.count}, () => false);
                this.loading = Array.from({length: this.count}, () => false);
            });

        },
        formatDate(date) {
            if( !(date instanceof Date) ){
                date = new Date(date);
            }
            let dateString = date.toDateString();

            let hours = date.getHours();
            let minutes = date.getMinutes();
            let ampm = hours >= 12 ? 'pm' : 'am';
            hours = hours % 12;
            hours = hours ? hours : 12; // the hour '0' should be '12'
            minutes = minutes < 10 ? '0'+minutes : minutes;
            let strTime = hours + ':' + minutes + ' ' + ampm;
            return `${dateString} - ${strTime}`;
        },
        badgeClass(status){
            switch(status) {
                case 'PENDING': return 'primary'
                case 'PROCESSING': return 'secondary'
                case 'READY': return 'info'
                case 'OUTFORDELIVERY': return 'warning'
                case 'DELIVERED': return 'success'
                case 'CANCELLED': return 'danger'
                default: return 'primary'
            }
        },
        changeStatus(status, index) {

            const order_id = this.tableData[index].order_id;

            // set loading
            this.$set(this.loading, index, true);

            this.$axios({
                method: 'patch',
                url: '/orders/status',
                data: {
                    order_id: order_id,
                    status: status
                },
            }).then((response) => {
                if (response.data && response.data.status === "success") {
                    this.$notify({
                        type: "success",
                        title: "Success",
                        message: "Status changed."
                    });
                } else {
                    this.$notify({
                        type: "danger",
                        title: "Something went Wrong!",
                        message: "Status not changed."
                    });
                }
            }).catch(() => {
                this.$notify({
                    type: "danger",
                    title: "Something went Wrong!",
                    message: "Status not changed."
                });
            }).finally(() => {
                this.$set(this.loading, index, false);
                this.$set(this.tableData[index], 'order_status', status);
            });
        }
    },
    mounted() {
        this.getOrders(this.storeId);

        // create a socket connection to server.
        const socket = io(this.baseUrl);

        // Show notifications on new orders.
        socket.on('new-order', () => {

            // Remove all existing notifications.
            this.$notifications.state.forEach((notification) => {
                this.$notifications.removeNotification(notification.timestamp);
            });

            // notify the user.
            this.$notify({
                type: 'default',
                title: 'New Orders are here..!',
                timeout: 480000, // 480 seconds
                message: 'Click here or refresh to see new orders.',
                icon: 'fa fa-concierge-bell',
                onClick: () => {
                    this.getOrders(this.storeId);
                }
            });

            // play the bell notification tone
            const notifAudio = new Audio('/audio/notification.ogg');
            // wrap it in async await IIFE
            (async function(){
                await notifAudio.play();
            })();
            
        });
    },
};
</script>
<style>
</style>
