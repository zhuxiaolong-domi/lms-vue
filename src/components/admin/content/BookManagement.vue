
<template>
  <div>
    <div style="margin-bottom: 16px">
      <el-row style="margin: 18px 0 0 18px ">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/admin/dashboard' }">管理中心</el-breadcrumb-item>
          <el-breadcrumb-item>用户管理</el-breadcrumb-item>
          <el-breadcrumb-item>用户信息</el-breadcrumb-item>
        </el-breadcrumb>
      </el-row>
      <a-button class="editable-add-btn" @click="handleAdd">
        添加图书信息
      </a-button>
      <span style="margin-right: 24px"></span>
      <a-button type="primary" :disabled="!hasSelected" :loading="loading" @click="start">
        重置页面
      </a-button>
      <span style="margin-left: 8px">
                <template v-if="hasSelected">
                    {{ `Selected ${selectedRowKeys.length} items` }}
                </template>
            </span>
    </div>
    <a-table
        :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        :columns="columns"
        :data-source="data"
        rowKey='id'
        bordered
    >
      <div
          slot="filterDropdown"
          slot-scope="{ setSelectedKeys, selectedKeys, confirm, clearFilters, column }"
          style="padding: 8px"
      >
        <a-input
            v-ant-ref="c => (searchInput = c)"
            :placeholder="`搜索 ${column.dataIndex}`"
            :value="selectedKeys[0]"
            style="width: 188px; margin-bottom: 8px; display: block;"
            @change="e => setSelectedKeys(e.target.value ? [e.target.value] : [])"
            @pressEnter="() => handleSearch(selectedKeys, confirm, column.dataIndex)"
        />
        <a-button
            type="primary"
            icon="search"
            size="small"
            style="width: 90px; margin-right: 8px"
            @click="() => handleSearch(selectedKeys, confirm, column.dataIndex)"
        >
          搜索
        </a-button>
        <a-button size="small" style="width: 90px" @click="() => handleReset(clearFilters)">
          重置
        </a-button>
      </div>
      <a-icon
          slot="filterIcon"
          slot-scope="filtered"
          type="search"
          :style="{ color: filtered ? '#108ee9' : undefined }"
      />
      <template slot="customRender" slot-scope="text, record, index, column">
            <span v-if="searchText && searchedColumn === column.dataIndex">
                <template
                    v-for="(fragment, i) in text.toString().split(new RegExp(`(?<=${searchText})|(?=${searchText})`, 'i'))"
                >
                <mark
                    v-if="fragment.toLowerCase() === searchText.toLowerCase()"
                    :key="i"
                    class="highlight"
                >{{ fragment }}</mark>
                <template v-else>{{ fragment }}</template>
                </template>
                </span>
        <template v-else>{{ text }}</template>
      </template>
      <template slot="operation_delete" slot-scope="text, record">
        <a-popconfirm
            v-if="data.length"
            title="确定要删除吗?"
            @confirm="() => onDelete(record.id)"
        >
          <a href="javascript:;">删除</a>
        </a-popconfirm>
      </template>
      <template
          v-for="col in ['id',  'bookname', 'author', 'date', 'press','abs']"
          :slot="col"
          slot-scope="text, record, index"
      >
        <div :key="col">
          <a-input
              v-if="record.editable"
              style="margin: -5px 0"
              :value="text"
              @change="e => handleChange(e.target.value, record.id, col)"
          />
          <template v-else>
            {{ text }}
          </template>
        </div>
      </template>
      <template slot="operation_edit" slot-scope="text, record, index">
        <div class="editable-row-operations">
                    <span v-if="record.editable">
                        <a @click="() => save(record.id)">Save</a>
                        <a-popconfirm title="Sure to cancel?" @confirm="() => cancel(record.id)">
                            <a>Cancel</a>
                        </a-popconfirm>
                    </span>
          <span v-else>
<!--                        <a :disabled="editingKey !== ''" @click="() => edit(record.id)">编辑</a>-->
                         <a :disabled="editingKey !== ''" @click="editNode(record)">编辑</a>
                    </span>
        </div>
      </template>
    </a-table>

    <el-dialog
        title="修改图书"
        :visible.sync="dialogFormVisible"
        @close="clear">
      <el-form v-model="form" style="text-align: left" ref="dataForm">
        <el-form-item label="书名" :label-width="formLabelWidth" prop="bookname">
          <el-input v-model="form.bookname" autocomplete="off" placeholder="不加《》"></el-input>
        </el-form-item>
        <el-form-item label="作者" :label-width="formLabelWidth" prop="author">
          <el-input v-model="form.author" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="出版日期" :label-width="formLabelWidth" prop="date">
          <el-input v-model="form.date" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="出版社" :label-width="formLabelWidth" prop="press">
          <el-input v-model="form.press" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="封面" :label-width="formLabelWidth" prop="cover">
          <el-input v-model="form.cover" autocomplete="off" placeholder="图片 URL"></el-input>
          <img-upload @onUpload="uploadImg" ref="imgUpload"></img-upload>
        </el-form-item>
        <el-form-item label="简介" :label-width="formLabelWidth" prop="abs">
          <el-input type="textarea" v-model="form.abs" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类" :label-width="formLabelWidth" prop="cid">
          <el-select v-model="form.category.id" placeholder="请选择分类" >
            <el-option label="文学" value="1"></el-option>
            <el-option label="流行" value="2"></el-option>
            <el-option label="文化" value="3"></el-option>
            <el-option label="生活" value="4"></el-option>
            <el-option label="经管" value="5"></el-option>
            <el-option label="科技" value="6"></el-option>
          </el-select>
        </el-form-item>
        <!--                <el-form-item label="状态" :label-width="formLabelWidth" prop="enabled">-->
        <!--                    <el-input v-model="form.enabled" autocomplete="off"></el-input>-->
        <!--                </el-form-item>-->
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="onSubmit">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>

const data = [];
let obj = {};
// import BulkRegistration from './BulkRegistration'
//2020.9.20 15.05 xiugaiqian
import ImgUpload from '../../library/ImgUpload'

export default {

  name: 'UserProfile',
  // components: {BulkRegistration},
  components: {ImgUpload},
  data() {
    this.cacheData = data.map(item => ({...item}));
    return {
      dialogFormVisible: false,
      dialogForm_id:0,
      dialogForm_cover:'',
      dialogForm_bookname:'',
      dialogForm_author:'',
      dialogForm_date:'',
      dialogForm_press:'',
      dialogForm_abs:'',
      dialogForm_cid:'',
      form: {
        id: '',
        bookname: '',
        author: '',
        date: '',
        press: '',
        cover: '',
        abs: '',
        category: {
          id: '',
          name: ''
        },
      },
      formLabelWidth: '120px',

      data,
      searchText: '',
      searchInput: null,
      searchedColumn: '',
      editingKey: '',
      columns: [
        {
          title: 'id',
          dataIndex: 'id',
          key: 'id',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'id',
          },
          onFilter: (value, record) =>
              record.id
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              }, 0);
            }
          },
        },
        {
          title: '书名',
          dataIndex: 'bookname',
          key: 'bookname',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'bookname',
          },
          onFilter: (value, record) =>
              record.bookname
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              });
            }
          },
        },
        {
          title: '作者',
          dataIndex: 'author',
          key: 'author',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'author',
          },
          onFilter: (value, record) =>
              record.author
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              });
            }
          },
        },
        {
          title: '出版日期',
          dataIndex: 'date',
          key: 'date',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'date',
          },
          onFilter: (value, record) =>
              record.date
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              });
            }
          },
        },
        {
          title: '出版社',
          dataIndex: 'press',
          key: 'press',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'press',
          },
          onFilter: (value, record) =>
              record.press
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              });
            }
          },
        },
        {
          title: '封面',
          dataIndex: 'cover',
          key: 'cover',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'cover',
          },
          onFilter: (value, record) =>
              record.cover
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              });
            }
          },
        },
        {
          title: '分类',
          dataIndex: 'cid',
          key: 'cid',
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'cid',
          },
          onFilter: (value, record) =>
              record.cid
                  .toString()
                  .toLowerCase()
                  .includes(value.toLowerCase()),
          onFilterDropdownVisibleChange: visible => {
            if (visible) {
              setTimeout(() => {
                this.searchInput.focus();
              });
            }
          },
        },
        {
          title: '删除',
          dataIndex: 'operation_delete',
          scopedSlots: {customRender: 'operation_delete'},
        },
        {
          title: '修改',
          dataIndex: 'operation_edit',
          scopedSlots: {customRender: 'operation_edit'},
        },
      ],
      selectedRowKeys: [], // Check here to configure the default column
      loading: false,
    };
  },
  computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0;
    },
  },
  mounted () {
    this.listBooks()
  },
  methods: {
    clear() {
      this.form = {
        id: '',
        bookname: '',
        author: '',
        date: '',
        press: '',
        cover: '',
        abs: '',
        category: {
          id: '',
          name: ''
        }
      }
      this.$refs.imgUpload.$refs.upload.clearFiles()
    },
    onSubmit() {
      this.$axios
          .post('/admin/content/books', {
            id: this.form.id,
            cover: this.form.cover,
            bookname: this.form.bookname,
            author: this.form.author,
            date: this.form.date,
            press: this.form.press,
            abs: this.form.abs,
            category: this.form.category
          }).then(resp => {
        if (resp && resp.status === 200) {
          console.log(resp.status)
          this.dialogFormVisible = false
          this.$emit('onSubmit')
          this.$message.success('修改成功')
        } else {
          console.log(resp.status)
          this.$message.error('提交错误')
        }
      })
          .catch(err => {
            this.$message.error('服务器错误')
          })
    },
    uploadImg () {
      this.form.cover = this.$refs.imgUpload.url
    },

    listBooks() {
      let _this = this
      this.$axios.get('/books').then(resp => {
        if (resp && resp.data.code === 200) {
          _this.data = resp.data.data
        }
      })
    },

    //demo
    start() {
      this.loading = true;
      // ajax request after empty completing
      setTimeout(() => {
        this.loading = false;
        this.selectedRowKeys = [];
      }, 500);
    },
    onSelectChange(selectedRowKeys) {
      console.log('selectedRowKeys changed: ', selectedRowKeys);
      this.selectedRowKeys = selectedRowKeys;
    },
    onDelete(id) {
      let _this = this
      const data = [...this.data];
      this.data = data.filter(item => item.id !== id);
      this.$axios.post('/admin/content/books/delete', {id: id}).then(resp=>{
        if (resp && resp.status === 200) {
          this.listUsers()
        }
      })
    },
    handleSearch(selectedKeys, confirm, dataIndex) {
      confirm();
      this.searchText = selectedKeys[0];
      this.searchedColumn = dataIndex;
    },

    handleReset(clearFilters) {
      clearFilters();
      this.searchText = '';
    },
    handleAdd() {
      const {count, data} = this;
      const newData = {
        key: count,
        id: `Edward King ${count}`,
        username: 32,
        name: `London, Park Lane no. ${count}`,
      };
      this.data = [...data, newData];
    },
    handleChange(value, id, column) {
      const newData = [...this.data];
      const target = newData.filter(item => id === item.id)[0];
      console.log('column ' + column + ' value ' + value)
      if (target) {
        target[column] = value;
        this.data = newData;
      }
    },
    edit(id) {
      const newData = [...this.data];
      const target = newData.filter(item => id === item.id)[0];
      this.editingKey = id;
      if (target) {
        target.editable = true;
        this.data = newData;
      }
    },
    save(id) {
      let _this = this
      const newData = [...this.data];
      const newCacheData = [...this.cacheData];
      const target = newData.filter(item => id === item.id)[0];
      const targetCache = newCacheData.filter(item => id === item.id)[0];
      if (target && targetCache) {
        delete target.editable;
        this.data = newData;
        Object.assign(targetCache, target);
        this.cacheData = newCacheData;
      }
      console.log("tar值 " + targetCache + " cache " + this.cacheData + " target " + target)
      this.editingKey = '';
      console.log(...this.data)
      this.$axios.put('/admin/user', {
        ...this.data
      })
          .then(resp => {
            if (resp && resp.data.code === 200) {
              this.$alert('用户信息修改成功')
              // 修改角色后重新请求用户信息，实现视图更新
              this.listUsers()
            } else {
              this.$alert(resp.data.message)
            }
          })
    },
    cancel(id) {
      const newData = [...this.data];
      const target = newData.filter(item => id === item.id)[0];
      this.editingKey = '';
      if (target) {
        Object.assign(target, this.cacheData.filter(item => id === item.id)[0]);
        delete target.editable;
        this.data = newData;
      }
    },
    editNode:function (item){
      console.log(item)
      this.dialogFormVisible = true
      this.dialogForm_id=item.id
      this.dialogForm_bookname=item.bookname
      this.dialogForm_author=item.author
      this.dialogForm_date=item.date
      this.dialogForm_press=item.press
      this.dialogForm_cover=item.cover
      this.dialogForm_abs=item.abs
      this.dialogForm_cid=item.cid
    }
  },
};
</script>

<style scoped>
.highlight {
  background-color: rgb(255, 192, 105);
  padding: 0;
}

.editable-row-operations a {
  margin-right: 8px;
}
</style>

