<template>
  <section class="rules">
    <Form ref="formInline" :model="formInline" inline>
      <FormItem label="业务" :label-width="40">
        <i-input v-model="formInline.app_name" placeholder="请输入业务名称" clearable />
      </FormItem>
      <FormItem label="异常来源" :label-width="70">
        <i-select
          v-model="formInline.exception_type"
          multiple
          clearable
          placeholder="请选择异常来源"
          style="width: 15vw;"
          :max-tag-count="2"
        >
          <Option v-for="(exception,index) in exceptionList" :key="index" :value="exception.value">
            {{ exception.name }}
          </Option>
        </i-select>
      </FormItem>
      <FormItem label="实例类型" :label-width="70">
        <i-select
          v-model="formInline.inst_type"
          multiple
          clearable
          placeholder="请选择实例类型"
          style="width: 12vw;"
        >
          <Option v-for="(type,index) in instTypeList" :key="index" :value="type.value">
            {{ type.name }}
          </Option>
        </i-select>
      </FormItem>
      <FormItem label="实例名称" :label-width="70">
        <i-input v-model="formInline.inst_name" placeholder="请输入实例名称" clearable />
      </FormItem>
      <FormItem>
        <Button type="success" class="mr-15" @click="search">查询</Button>
      </FormItem>
    </Form>
    <CwTable
      :columns="columns"
      :data="data"
      :loading="loading"
      :current="page.currentPage"
      :total="page.total"
      :page-size="page.pageSize"
      @size-change="sizeChange"
      @current-change="currentChange"
    >
      <template slot="instNameIP" slot-scope="{row}">
        <Tooltip max-width="200" class="w-100" :content="row.inst_name+'/'+row.inst_ip" placement="bottom" transfer>
          <div class="ellipse">{{ row.inst_name+'/'+row.inst_ip }}</div>
        </Tooltip>
      </template>
      <template slot="menu" slot-scope="{row}">
        <Button type="primary" size="small" @click="confirm(row)">取消忽略</Button>
      </template>
    </CwTable>
  </section>
</template>

<script>
import CwTable from '@/components/CwTable/CwTable'
export default {
  name: 'CloseException',
  components: { CwTable },
  data() {
    return {
      exceptionList: [{
        name: '告警',
        value: 'alert'
      }, {
        name: '巡检',
        value: 'check'
      }, {
        name: '备份',
        value: 'backup'
      }, {
        name: '补丁',
        value: 'patch'
      }, {
        name: '基线',
        value: 'baseline'
      }, {
        name: '证书',
        value: 'ssl'
      }],
      instTypeList: [{
        name: '主机',
        value: 'host'
      }, {
        name: 'MySQL',
        value: 'mysql'
      }],
      formInline: {
        app_name: '',
        exception_type: [],
        inst_type: [],
        inst_name: ''
      },
      loading: false,
      columns: [
        {
          title: '业务',
          key: 'app_name',
          align: 'center'
        }, {
          title: '异常来源',
          key: 'exception_type_cn',
          align: 'center',
          width: '120px',
          tooltip: true
        }, {
          title: '实例类型',
          key: 'inst_type_cn',
          align: 'center'
        }, {
          title: '实例名称/IP',
          slot: 'instNameIP',
          width: '200px',
          align: 'center'
        }, {
          title: '异常描述',
          key: 'exception_describe',
          tooltip: true,
          align: 'center'
        }, {
          title: '忽略时长',
          key: 'ignore_type_cn',
          align: 'center'
        }, {
          title: '取消忽略日期',
          key: 'ignore_date',
          align: 'center',
          width: '150px'
        }, {
          title: '操作',
          slot: 'menu',
          align: 'center',
          width: '100px'
        }
      ],
      data: [],
      page: {
        currentPage: 1,
        total: 0,
        pageSize: 10
      },
      drawerVisible: false
    }
  },
  created() {
    this.query().then()
  },
  methods: {
    async search() {
      await this.query()
    },
    async query() {
      this.loading = true
      try {
        const res = await this.$api.show.getRules({
          params: this.formInline,
          num: this.page.pageSize,
          page: this.page.currentPage
        })
        this.loading = false
        if (res.result) {
          this.page.total = res.data.total
          this.data = res.data.data
        } else {
          this.$CwMessage('error', res.message)
        }
      } catch (e) {
        this.loading = false
        this.$CwMessage('error', e)
      }
    },
    confirm(row) {
      this.$CWConfirm({
        type: 'primary',
        title: '取消忽略',
        content: '确定要取消这条规则的忽略吗',
        okText: '确定',
        cancelText: '取消'
      }).then(async() => {
        try {
          const res = await this.$api.show.cancelIgnore({ id: row.id })
          if (res.result) {
            this.$CwMessage('success', '取消忽略成功')
            await this.query()
          } else {
            this.$CwMessage('error', res.message)
          }
        } catch (e) {
          this.$CwMessage('error', e)
        }
      }).catch(() => {

      })
    },
    async create() {},
    sizeChange(val) {
      this.page.currentPage = 1
      this.page.pageSize = val
      this.query().then()
    },
    currentChange(val) {
      this.page.currentPage = val
      this.query().then()
    }
  }
}
</script>

<style scoped>

</style>
