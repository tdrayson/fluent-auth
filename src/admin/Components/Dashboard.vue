<template>
    <div class="dashboard box_wrapper">
        <div class="box_narrow">
            <div style="font-size: 16px; margin-bottom: 20px;">
                {{$t('Hello %s, view your security config and recent login activities', me.full_name)}}
            </div>

            <quick-stat-bar />

            <el-row :gutter="30">
                <el-col :md="12" :sm="24">
                    <div class="box dashboard_box">
                        <div class="box_header" style="padding: 10px 15px;font-weight: normal; font-size: 16px;">
                            {{$t('Recent Failed & Blocked Logins')}}
                        </div>
                        <div style="padding: 0;" class="box_body">
                            <div style="padding: 15px;" v-if="fetching_failed_logs">
                                <el-skeleton :animated="true"></el-skeleton>
                            </div>
                            <log-table v-else-if="failed_logs.length" :logs="failed_logs" />
                            <div v-else style="padding: 15px">
                                <el-empty :description="$t('Not enough data. This section will show recent failed login attempts')" />
                            </div>
                        </div>
                    </div>
                    <div class="box dashboard_box">
                        <div class="box_header" style="padding: 10px 15px;font-weight: normal; font-size: 16px;">
                            {{$t('Settings Overview')}}
                            <div class="box_actions">
                                <span @click="$router.push({ name: 'settings' })" style="cursor: pointer" :title="$t('Go to Settings')" class="dashicons dashicons-admin-settings"></span>
                            </div>
                        </div>
                        <div style="padding: 0;" class="box_body">
                            <ul class="fls_listed_data">
                                <li>
                                    <span class="fls_label">{{$t('Disable XML-RPC Requests')}}</span>
                                    <span class="fls_value">{{settings.disable_xmlrpc}}</span>
                                </li>
                                <li>
                                    <span class="fls_label">{{$t('Disable Rest Remote App Login')}}</span>
                                    <span class="fls_value">{{settings.disable_app_login}}</span>
                                </li>
                                <li>
                                    <span class="fls_label">{{$t('Log Login Logs')}}</span>
                                    <span class="fls_value">{{settings.enable_auth_logs}}</span>
                                </li>
                                <li>
                                    <span class="fls_label">{{$t('Disable Public User Indexing')}}</span>
                                    <span class="fls_value">{{settings.disable_users_rest}}</span>
                                </li>
                                <li>
                                    <span class="fls_label">{{$t('Login Notifications')}}</span>
                                    <span class="fls_value">{{(settings.notification_user_roles.length && settings.notification_email) ? 'yes' : 'no'}}</span>
                                </li>
                            </ul>
                        </div>
                    </div>
                </el-col>
                <el-col :md="12" :sm="24">
                    <div class="box dashboard_box">
                        <div class="box_header" style="padding: 10px 15px;font-weight: normal; font-size: 16px;">
                            {{$t('Recent Successful Logins')}}
                        </div>
                        <div style="padding: 0;" class="box_body">
                            <div style="padding: 15px;" v-if="fetching_failed_logs">
                                <el-skeleton :animated="true"></el-skeleton>
                            </div>
                            <log-table v-else-if="success_logs.length" :logs="success_logs" />
                            <div v-else style="padding: 15px">
                                <el-empty :description="$t('Not enough data. This section will show recent successful logins')" />
                            </div>
                        </div>
                    </div>
                </el-col>
            </el-row>
        </div>
    </div>
</template>

<script type="text/babel">
import LogTable from './_LogTable';
import QuickStatBar from './stat/_QuickStatBar';

export default {
    name: 'Dashboard',
    components: {
        LogTable,
        QuickStatBar
    },
    data() {
        return {
            me: this.appVars.me,
            failed_logs: [],
            success_logs: [],
            fetching_failed_logs: false,
            fetching_success_logs: false,
            settings: this.appVars.auth_settings
        }
    },
    methods: {
        fetchLogs(type, statuses, perPage = 10) {
            this['fetching_'+type] = true;
            this.$get('auth-logs', {
                per_page: perPage,
                statuses: statuses,
                page: 1
            })
                .then(response => {
                    this[type] = response.logs.data;
                })
                .catch((errors) => {
                    this.$handleError(errors);
                })
                .finally(() => {
                    this['fetching_'+type] = false;
                });
        }
    },
    mounted() {
        this.fetchLogs('failed_logs', ['failed', 'blocked']);
        this.fetchLogs('success_logs', ['success']);
    }
};
</script>
