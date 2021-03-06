<template>
  <div class="u-page__product">
    <div class="u-productAdd">
      <h1 class="u-title">
        <span class="u-index">1</span>商品基本信息
      </h1>
      <el-form ref="ruleForm" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="商品名称" prop="name">
          <el-input
            v-model="form.name"
            size="small"
            style="width: 400px"
            placeholder="商品标题组成：商品描述+规格"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品品牌">
          <el-input v-model="form.brand" size="small" style="width: 400px" placeholder="请填写品牌"></el-input>
        </el-form-item>
        <el-form-item label="商品产地">
          <el-input v-model="form.address" size="small" style="width: 400px" placeholder="请填写产地"></el-input>
        </el-form-item>
        <el-form-item label="适用对象" prop="petType">
          <el-select v-model="form.petType" clearable placeholder="请选择" size="small">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品分类" prop="categoryId">
          <el-select v-model="form.categoryId" clearable placeholder="请选择" size="small">
            <el-option
              v-for="item in categoryOptions"
              :key="item.categoryId"
              :label="item.categoryName"
              :value="item.categoryId"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品描述">
          <el-input
            v-model="form.detail"
            rows="5"
            type="textarea"
            style="width: 400px"
            placeholder="建议300字以内，未填写讲根据商品标题自动生成"
          ></el-input>
        </el-form-item>
        <el-form-item label="商品轮播" prop="productImgUrl">
          <el-upload
            :action="uploadUrl"
            list-type="picture-card"
            :on-preview="handleBannerPreview"
            :on-remove="handleBannerRemove"
            :on-success="handleBannerSuccess"
            :file-list="fileList"
          >
            <i class="el-icon-plus"></i>
          </el-upload>
          <el-dialog :visible.sync="dialogVisible">
            <img width="100%" :src="dialogImageUrl" alt />
          </el-dialog>
          <div class="u-tip">请至少上传一张轮播图片，推荐尺寸400x400</div>
        </el-form-item>
      </el-form>

      <h1 class="u-title">
        <span class="u-index">2</span>商品规格与库存
      </h1>

      <el-form ref="form" label-width="80px">
        <el-form-item label="商品规格">
          <div v-show="specs.length" class="u-specs__tip">最多添加两个商品规格</div>
          <div v-for="(sp, index) in specs" :key="index" class="u-specs__item">
            <div class="u-specs__header">
              <el-select
                v-model="sp.type"
                size="small"
                :placeholder="`规格${index + 1}`"
                @change="handleTypeChagne(sp, $event)"
              >
                <el-option
                  v-for="item in getOptions(index)"
                  :key="item.type"
                  :label="item.name"
                  :value="item.type"
                ></el-option>
              </el-select>
              <el-button type="text" size="small" @click="deleteSpecs(index)">删除规格</el-button>
            </div>
            <div v-if="sp.type" class="u-enum">
              <div class="u-enum__list">
                <el-tag
                  v-for="x in sp.val"
                  :key="x"
                  class="u-enum__item"
                  closable
                  :disable-transitions="false"
                  type="info"
                  @close="handleEnumClose(sp, x)"
                >{{ x }}</el-tag>
              </div>
              <div>
                <el-input v-model.trim="sp.enum" size="mini" style="width: 100px"></el-input>
                <el-button type="text" size="mini" @click="handleAddEnum(sp)">添加</el-button>
              </div>
            </div>
          </div>
          <el-button
            v-show="specs.length < 2"
            icon="el-icon-plus"
            size="small"
            @click="addSpecs"
          >添加规格</el-button>
        </el-form-item>
      </el-form>

      <h1 class="u-title">
        <span class="u-index">3</span>价格及库存
      </h1>

      <el-table :data="skuList" style="width: 100%">
        <!-- 单规格 -->
        <template v-if="effectSpecs.length === 1">
          <el-table-column :label="specs[0].name" width="100">
            <template slot-scope="{row}">
              <span>{{ row.specs[0].value }}</span>
            </template>
          </el-table-column>
        </template>
        <!-- 双规格 -->
        <template v-if="effectSpecs.length === 2">
          <el-table-column :label="specs[0].name" width="100">
            <template slot-scope="{row}">
              <span>{{ row.specs[0].value }}</span>
            </template>
          </el-table-column>
          <el-table-column :label="specs[1].name" width="100">
            <template slot-scope="{row}">
              <span>{{ row.specs[1].value }}</span>
            </template>
          </el-table-column>
        </template>
        <el-table-column label-class-name="u-require" label="sku名称" width="200">
          <template slot-scope="{row}">
            <el-input v-model="row.skuName" size="small"></el-input>
          </template>
        </el-table-column>
        <el-table-column label-class-name="u-require" label="sku价格" width="160" align="center">
          <template slot-scope="{row}">
            <el-input-number
              v-model="row.price"
              controls-position="right"
              :min="0"
              :max="99999"
              size="small"
            ></el-input-number>
          </template>
        </el-table-column>
        <el-table-column label-class-name="u-require" label="sku库存" width="160" align="center">
          <template slot-scope="{row}">
            <el-input-number
              v-model="row.stock"
              controls-position="right"
              :min="0"
              :max="99999"
              size="small"
            ></el-input-number>
          </template>
        </el-table-column>
        <el-table-column label-class-name="u-require" label="sku小图(400x400)" align="center">
          <template slot-scope="{row}">
            <el-upload
              :action="uploadUrl"
              :show-file-list="false"
              :on-success="handleSmallSuccess.bind(this, row)"
              :before-upload="beforeUpload"
            >
              <el-button v-if="!row.skuImgUrl" type="primary" size="mini">
                上传图片
                <i class="el-icon-upload el-icon--right"></i>
              </el-button>
              <el-image
                v-else
                style="width: 50px; height: 50px"
                :src="row.skuImgUrl"
                fit="fill"
                lazy
                webp
              />
            </el-upload>
          </template>
        </el-table-column>
        <el-table-column label-class-name="u-require" label="sku详情图(750x2000)" align="center">
          <template slot-scope="{row}">
            <el-upload
              :action="uploadUrl"
              :show-file-list="false"
              :on-success="handleDetailSuccess.bind(this, row)"
              :before-upload="beforeUpload"
            >
              <el-button v-if="!row.skuDetailImgUrl" type="primary" size="mini">
                上传图片
                <i class="el-icon-upload el-icon--right"></i>
              </el-button>
              <el-image
                v-else
                style="width: 50px; height: 50px"
                :src="row.skuDetailImgUrl"
                fit="fill"
                lazy
                webp
              />
            </el-upload>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="u-submit">
      <el-button :loading="submitLoading" type="primary" @click="handleSubmit">{{ $route.query.edit ? '更新商品': '提交发布' }}</el-button>
      <!-- <el-button>保存草稿</el-button> -->
    </div>
  </div>
</template>

<script>
import _ from 'lodash'
import store from 'store'
import eventbus from '@/lib/eventbus'
import * as productApi from '@/api/product'

const specsList = [
  { type: 'weight', name: '重量' },
  { type: 'shap', name: '形状' },
  { type: 'size', name: '尺寸' },
  { type: 'package', name: '套餐' },
  { type: 'taste', name: '口味' }
  // { type: 'taste', name: '口味' },
  // { type: 'weight', name: '重量' },
  // { type: 'color', name: '颜色' },
  // { type: 'size', name: '尺寸' },
  // { type: 'type', name: '型号' },
  // { type: 'style', name: '款式' },
  // { type: 'material', name: '材质' },
  // { type: 'group', name: '适用群体' },
  // { type: 'volume', name: '容量' },
  // { type: 'scent', name: '香型' },
  // { type: 'meal', name: '套餐' }
]
const specsMap = {}
specsList.forEach(item => {
  specsMap[item.type] = item
})
const genSku = function() {
  return {
    skuName: '',
    skuImgUrl: '',
    skuDetailImgUrl: '',
    price: '',
    stock: '',
    specs: null
  }
}
export default {
  data() {
    var validateBanner = (rule, value, callback) => {
      if (!_.get(value, 'length')) {
        callback(new Error('至少上传一张图片'))
      } else {
        callback()
      }
    }
    return {
      doingSave: false,
      submitLoading: false,
      enum1: '',
      enum2: '',
      specsList,
      specsMap,
      form: {
        categoryId: '',
        petType: '',
        name: '',
        brand: '',
        address: '',
        detail: '',
        productImgUrl: []
      },
      specs: [],
      skuList: [],
      dialogVisible: false,
      dialogImageUrl: '',
      fileList: [],
      options: [
        { label: '狗狗', value: 1 },
        { label: '猫咪', value: 2 }
      ],
      categoryOptions: [],
      rules: {
        categoryId: {
          required: true,
          message: '请选择商品分类',
          trigger: 'change'
        },
        petType: {
          required: true,
          message: '请选择适用对象',
          trigger: 'change'
        },
        name: { required: true, message: '请输入名称', trigger: 'blur' },
        productImgUrl: {
          required: true,
          validator: validateBanner,
          trigger: 'change'
        }
      }
    }
  },
  computed: {
    effectSpecs() {
      return this.specs.filter(item => !!item.val.length)
    }
  },
  watch: {
    fileList(val) {
      this.form.productImgUrl = val.map(item => {
        return {
          imgUrl: item.url
        }
      })
      this.$nextTick(() => {
        this.$refs.ruleForm.validateField('productImgUrl')
      })
    }
  },
  mounted() {
    this.init()
    this.genSku()
    this.queryProductCategory()
  },
  // 拦截确认
  beforeRouteLeave(to, from, next) {
    next()
  },
  methods: {
    init() {
      if (this.$route.query.edit) {
        // 编辑商品
        this.recoverData()
      } else {
        // 新增商品
      }
    },
    recoverData() {
      const historyData = store.get('product_edit')
      const { product, productSkuList, productBannerImgList } = historyData
      this.form = {
        categoryId: product.categoryId,
        petType: product.petType,
        name: product.name,
        brand: product.brand,
        address: product.address,
        detail: product.detail,
        productImgUrl: []
      }
      this.specs = product.specs || []
      this.skuList = productSkuList
      this.fileList = productBannerImgList.map(item => {
        return {
          url: item.imgUrl
        }
      })
    },
    handleSubmit() {
      this.$refs['ruleForm'].validate(valid => {
        if (valid) {
          this.checkSku()
            .then(() => {
              const params = this.genParams()
              if (this.submitLoading) {
                return
              }
              this.submitLoading = true
              if (this.$route.query.edit) {
                productApi
                  .updateProduct(params)
                  .then(res => {
                    this.$message.success('更新商品成功')
                    this.submitLoading = false
                    eventbus.$emit('closeCurrenPage')
                  })
                  .catch(err => {
                    this.submitLoading = false
                    this.$message.error(err.message)
                  })
              } else {
                productApi
                  .insertProduct(params)
                  .then(res => {
                    this.$message.success('新增商品成功')
                    this.submitLoading = false
                    eventbus.$emit('refreshCurrenPage')
                  })
                  .catch(err => {
                    this.submitLoading = false
                    this.$message.error(err.message)
                  })
              }
            })
            .catch(err => {
              this.$message.error(err.message)
            })
        } else {
          return false
        }
      })
    },
    // 生成提交参数
    genParams() {
      const params = {
        ...this.form,
        specList: this.specs
          .filter(item => !!_.get(item, 'val.length'))
          .map(item => ({ name: item.name, type: item.type, val: item.val })),
        skuInfoList: this.skuList
      }
      if (this.$route.query.edit) {
        const historyData = store.get('product_edit')
        const { product } = historyData
        params.id = product.id
        params.productId = product.productId
      }
      return params
    },
    // 检测sku是否符合规范
    checkSku() {
      return new Promise((resolve, reject) => {
        const skuList = this.skuList
        for (let i = 0; i < skuList.length; i++) {
          const sku = skuList[i]
          if (
            sku.skuName === '' ||
            sku.skuImgUrl === '' ||
            sku.skuDetailImgUrl === '' ||
            sku.price === '' ||
            sku.stock === ''
          ) {
            reject(new Error(`第${i + 1}个商品内容存在空情况，请补充完整`))
            return
          }
        }
        resolve()
      })
    },
    handleSmallSuccess(row, res, file) {
      row.skuImgUrl = res.data
    },
    handleDetailSuccess(row, res, file) {
      row.skuDetailImgUrl = res.data
    },
    beforeUpload() {
      return true
    },
    queryProductCategory() {
      productApi.queryProductCategory().then(res => {
        this.categoryOptions = res || []
      })
    },
    getOptions(index) {
      const x = index === 0 ? 1 : 0
      const exist = this.specs[x]
      if (exist && exist.type) {
        return this.specsList.filter(item => item.type !== exist.type)
      }
      return this.specsList
    },
    // 规格发生改变，规格数发生改变，都需要刷新Sku
    genSku() {
      const specs = this.specs.filter(item => !!item.val.length)
      const skuList = []
      if (specs.length === 0) {
        skuList.push(genSku())
      }
      if (specs.length === 1) {
        specs[0].val.forEach(v => {
          const s = {
            type: specs[0].type,
            name: specs[0].name,
            value: v
          }
          const sku = genSku()
          sku.specs = [s]
          skuList.push(sku)
        })
      }
      if (specs.length === 2) {
        specs[0].val.forEach(v => {
          const s1 = {
            type: specs[0].type,
            name: specs[0].name,
            value: v
          }
          specs[1].val.forEach(v2 => {
            const s2 = {
              type: specs[1].type,
              name: specs[1].name,
              value: v2
            }
            const sku = genSku()
            sku.specs = [s1, s2]
            skuList.push(sku)
          })
        })
      }
      // TODO: 原数据填充
      skuList.forEach(sku => {
        this.skuList.forEach(oldSku => {
          if (this.isSameSpeces(sku.specs || [], oldSku.specs || [])) {
            Object.assign(sku, oldSku)
          }
        })
      })
      this.skuList = skuList
    },
    // 判断老sku规格是否和最新的一样
    isSameSpeces(specs, oldSpecs) {
      if (oldSpecs.length !== specs.length) {
        return false
      }
      const diff = _.differenceWith(specs, oldSpecs, (a, b) => {
        return a.type === b.type && a.value === b.value
      })
      return diff.length === 0
    },
    handleEnumClose(specs, val) {
      specs.val = specs.val.filter(i => i !== val)
      this.genSku()
    },
    handleAddEnum(specs) {
      if (!specs.enum) return
      specs.val = _.uniq([...specs.val, specs.enum])
      specs.enum = ''
      this.genSku()
    },
    handleTypeChagne(a, b) {
      a.name = this.specsMap[b].name
    },
    addSpecs() {
      this.specs.push({ type: '', name: '', val: [], enum: '' })
    },
    deleteSpecs(index) {
      this.specs.splice(index, 1)
      this.genSku()
    },
    handleBannerSuccess(res, file, fileList) {
      file.url = res.data
      this.fileList = fileList
    },
    handleBannerRemove(file, fileList) {
      this.fileList = fileList
    },
    handleBannerPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    }
  }
}
</script>

<style lang="less" scoped>
.u-page__product {
  padding-bottom: 400px;
}
.u-productAdd {
  width: 1040px;
  padding: 20px;
  border: 1px solid #ccc;
}
.u-title {
  font-size: 16px;
  font-weight: 500;
}
.u-index {
  margin-right: 5px;
  font-size: 30px;
  color: #409eff;
}
.u-tip {
  color: #999;
}
.u-specs {
  &__tip {
    color: #999;
  }
  &__header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 5px 20px;
    background: #f4f5f9;
  }
  &__item {
    margin-bottom: 10px;
  }
}
.u-enum {
  padding: 20px;
  background: #fbfbfb;
  &__list {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
  }
  &__item {
    text-align: center;
    margin-right: 10px;
    margin-bottom: 10px;
  }
}
.u-submit {
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: 100;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100px;
  box-shadow: 1px 1px 2px 2px #eee;
  background: #fff;
}
</style>
<style lang="less">
.u-productAdd {
  .u-require.cell {
    &::before {
      content: '*';
      color: red;
    }
  }
}
</style>
