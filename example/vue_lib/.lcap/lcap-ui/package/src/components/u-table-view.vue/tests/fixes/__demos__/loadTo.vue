<!-- 后端分页 -->
<template>
    <u-linear-layout direction="vertical">
        <u-table-view :data-source="load" pagination :page-size="pageSize" :page-number="pageNumber" ref="tableview" :showJumper="true">
            <u-table-view-column title="用户名" field="name" width="15%"></u-table-view-column>
            <u-table-view-column title="手机号码" field="phone" width="20%"></u-table-view-column>
            <u-table-view-column title="地址" field="address"></u-table-view-column>
            <u-table-view-column title="最近登录时间" field="loginTime" formatter="placeholder | date" width="20%"></u-table-view-column>
        </u-table-view>
    </u-linear-layout>
    </template>
    <script>
    // 模拟后端请求
    const mockRequest = (data, timeout = 10) => new Promise((res, rej) => setTimeout(() => res(data), timeout));
    // 模拟构造 75 条后端数据
    const mockData = (() => {
        const baseData = [
            { name: '张三', phone: '18612917895', email: 'zhangsan@163.com', address: '浙江省杭州市滨江区网商路599号网易大厦', createdTime: 1464421931000, loginTime: 1527515531000 },
            { name: '小明', phone: '13727160283', email: 'xiaoming@163.com', address: '浙江省杭州市滨江区江虹路459号英飞特科技园', createdTime: 1520864676000, loginTime: 1552400676000 },
            { name: '李四', phone: '18897127809', email: 'lisi@163.com', address: '浙江省杭州市滨江区秋溢路606号西可科技园', createdTime: 1494488730000, loginTime: 1558165530000 },
            { name: '李华', phone: '18749261214', email: 'lihua@163.com', address: '浙江省杭州市滨江区长河路590号东忠科技园', createdTime: 1476073921000, loginTime: 1544428081000 },
            { name: '王五', phone: '13579340020', email: 'wangwu@163.com', address: '浙江省杭州市滨江区网商路599号网易大厦二期', createdTime: 1468614726000, loginTime: 1531675926000 },
        ];
    
        const result = [];
        for (let i = 0; i < 75; i++) {
            const item = Object.assign({}, baseData[i % 5]);
            item.name += '-' + (Math.random() * 20 >> 0);
            item.phone = String(+item.phone + (Math.random() * 10 >> 0) * Math.pow(10, Math.random() * 8 >> 0));
            item.createdTime += i * 1000 * 3600 * 24;
            item.loginTime += i * 1000 * 3600 * 24;
            result.push(item);
        }
    
        return result;
    })();
    // 模拟数据服务
    const mockService = {
        loadList(offset, limit) {
            // 在这里模拟了一个后端分页的请求
            return mockRequest({
                total: mockData.length,
                data: mockData.slice(offset, offset + limit),
            });
        },
    };
    
    export default {
        data() {
            return {
                result: undefined,
                pageSize: 10,
                pageNumber: 1,
                pageTo: null,
                data: mockData,
            };
        },
        methods: {
            load({ paging }) {
                return mockService.loadList(paging.offset, paging.limit)
                    .then((result) => this.result = result); // 这句只是在 Demo 中打印一下数据，方便查看
            },
            loadTo() {
                this.$refs.tableview.loadTo(this.pageTo);
            },
        },
    };
    </script>