<template>
    <div class="dashboard-container">
        <div class="app-container">
            <page-tools :show-before="true">
                <span slot="before">共166条记录</span>
                <template slot="after">
                    <el-button size="small" type="warning" @click="$router.push('/import?type=user')">导入</el-button>
                    <el-button size="small" type="danger">导出</el-button>
                    <el-button size="small" type="primary" @click="showDialog = true">新增员工</el-button>
                </template>
            </page-tools>
            <!-- 放置表格和分页 -->
            <el-card v-loading="loading">
                <el-table border :data="list">
                    <el-table-column label="序号" sortable="" type="index" />
                    <el-table-column label="姓名" sortable="" prop="username" />
                    <el-table-column label="头像" align="center">
                        <template slot-scope="{ row }">
                            <img
                                slot="reference"
                                v-imageerror="require('@/assets/common/bigUserHeader.png')"
                                :src="row.staffPhoto"
                                style="border-radius: 50%; width: 100px; height: 100px; padding: 10px"
                                alt=""
                                @click="showQrCode(row.staffPhoto)"
                            />
                        </template>
                    </el-table-column>
                    <el-table-column label="工号" sortable="" prop="workNumber" />
                    <el-table-column
                        label="聘用形式"
                        sortable=""
                        prop="formOfEmployment"
                        :formatter="formatEmployment"
                    />
                    <el-table-column label="部门" sortable="" prop="departmentName" />
                    <el-table-column label="入职时间" sortable="" prop="timeOfEntry">
                        <template slot-scope="row">
                            {{ row.row.timeOfEntry | formatDate }}
                        </template>
                    </el-table-column>
                    <el-table-column label="账户状态" align="center" sortable="" prop="enableState">
                        <template slot-scope="{ row }">
                            <!-- 根据当前状态来确定 是否打开开关 -->
                            <el-switch :value="row.enableState === 1" />
                        </template>
                    </el-table-column>
                    <el-table-column label="操作" sortable="" fixed="right" width="280">
                        <template slot-scope="{ row }">
                            <el-button type="text" size="small" @click="$router.push(`/employees/detail/${row.id}`)"
                                >查看</el-button
                            >
                            <el-button type="text" size="small">转正</el-button>
                            <el-button type="text" size="small">调岗</el-button>
                            <el-button type="text" size="small">离职</el-button>
                            <el-button type="text" size="small">角色</el-button>
                            <el-button type="text" size="small" @click="deleteEmployee(row.id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <!-- 分页组件 -->
                <el-row type="flex" justify="center" align="middle" style="height: 60px">
                    <el-pagination
                        layout="prev, pager, next"
                        :page-size="page.size"
                        :current-page="page.page"
                        :total="page.total"
                        @current-change="changePage"
                    />
                </el-row>
            </el-card>

            <el-dialog title="二维码" :visible.sync="showCodeDialog">
                <el-row type="flex" justify="center">
                    <canvas ref="myCanvas" />
                </el-row>
            </el-dialog>

            <!-- 放置新增组件 -->
            <add-employee :show-dialog.sync="showDialog" />
        </div>
    </div>
</template>
<script>
import { getEmployeeList, delEmployee } from "@/api/employees"
import EmployeeEnum from "@/api/constant/employees"
import AddEmployee from "./components/add-employee"
import QrCode from "qrcode"
export default {
    components: {
        AddEmployee,
    },
    data() {
        return {
            loading: false,
            list: [], // 接数据的
            page: {
                page: 1, // 当前页码
                size: 10,
                total: 0, // 总数
            },
            showDialog: false,
            showCodeDialog: false,
        }
    },
    created() {
        this.getEmployeeList()
    },
    methods: {
        changePage(newPage) {
            this.page.page = newPage
            this.getEmployeeList()
        },
        async getEmployeeList() {
            this.loading = true
            const { total, rows } = await getEmployeeList(this.page)

            this.page.total = total
            this.list = rows
            this.loading = false
        },
        // 格式化聘用形式
        formatEmployment(row, column, cellValue, index) {
            // 要去找 1所对应的值
            const obj = EmployeeEnum.hireType.find(item => item.id === cellValue)
            return obj ? obj.value : "未知"
        },

        // 删除员工
        async deleteEmployee(id) {
            try {
                await this.$confirm("您确定删除该员工吗")
                await delEmployee(id)
                this.getEmployeeList()
                this.$message.success("删除员工成功")
            } catch (error) {
                console.log(error)
            }
        },
        showQrCode(url) {
            // url存在的情况下 才弹出层
            if (url) {
                this.showCodeDialog = true // 数据更新了 但是我的弹层会立刻出现吗 ？页面的渲染是异步的！！！！
                // 有一个方法可以在上一次数据更新完毕，页面渲染完毕之后
                this.$nextTick(() => {
                    // 此时可以确认已经有ref对象了
                    QrCode.toCanvas(this.$refs.myCanvas, url) // 将地址转化成二维码
                    // 如果转化的二维码后面信息 是一个地址的话 就会跳转到该地址 如果不是地址就会显示内容
                })
            } else {
                this.$message.warning("该用户还未上传头像")
            }
        },
    },
}
</script>

<style></style>
