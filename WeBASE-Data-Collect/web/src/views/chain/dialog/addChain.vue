<template>
    <div>
        <el-dialog :title="title" :visible.sync="dialogVisible" :before-close="modelClose" class="dialog-wrapper" width="400px" :center="true" :show-close='true'>
            <div>
                <el-form :model="chainFrom" :rules="rules" ref="chainFrom" label-width="100px" class="demo-ruleForm">
                    <el-form-item label="区块链编号" prop="chainId" style="width:330px">
                        <el-input v-model.trim="chainFrom.chainId" :disabled="chainFrom['disabled']"></el-input>
                    </el-form-item>
                    <el-form-item label="区块链名称" prop="chainName" style="width:330px">
                        <el-input v-model.trim="chainFrom.chainName" :clearable="true"></el-input>
                    </el-form-item>

                    <el-form-item label="区块链类型" prop="type" style="width:330px" class="chain-type">
                        <el-select v-model="chainFrom.type" placeholder="请选择" :disabled="chainFrom['mDisabled']">
                            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="区块链备注" prop="description" style="width:330px">
                        <el-input v-model.trim="chainFrom.description" type="textarea"></el-input>
                    </el-form-item>
                </el-form>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button @click="modelClose">取 消</el-button>
                <el-button type="primary" :loading="loading" @click="submit('chainFrom')">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import { addChain, modifyChain } from "@/util/api"
export default {
    name: "addChain",
    props: ['show', 'chainDialogTitle', 'chainDialogOptions'],
    watch: {
        "chainDialogOptions.type": {
            handler(val) {
                this.dialogType = val;
                switch (val) {
                    case "creat":
                        this.chainFrom = {
                            chainName: "",
                            chainId: "",
                            type: 0,
                            description: "",
                            disabled: false,
                            mDisabled: false,
                            dShow: true,
                            mShow: true,
                        };
                        break;
                    case "modify":
                        this.chainFrom = {
                            chainName: this.chainDialogOptions.data["chainName"],
                            chainId: this.chainDialogOptions.data["chainId"],
                            type: this.chainDialogOptions.data["encryptType"],
                            description: this.chainDialogOptions.data["description"],
                            disabled: true,
                            mDisabled: true,
                            dShow: true,
                            mShow: false,
                        };
                        break;
                }
            },
            deep: true,
            immediate: true
        }
    },
    data() {
        return {
            dialogVisible: this.show,
            title: this.chainDialogTitle,
            chainFrom: {},
            dialogType: this.chainDialogOptions.type,
            loading: false,
            rules: {
                chainName: [
                    {
                        required: true,
                        message: "请输入区块链名称",
                        trigger: "blur"
                    },
                    {
                        pattern: /^.{0,10}$/,
                        message: "区块链名称长度不超过10位",
                        trigger: "blur"
                    }
                ],
                chainId: [
                    {
                        required: true,
                        message: "请输入区块链编号",
                        trigger: "blur"
                    },
                    {
                        pattern: /^\d{0,3}$/,
                        message: "区块链编号仅允许数字,长度不超过3位",
                        trigger: "blur"
                    }
                ],
                type: [
                    {
                        required: true,
                        message: "请选择区块链类型",
                        trigger: "blur"
                    }
                ],
                description: [
                    {
                        required: true,
                        message: "请选择输入区块链描述",
                        trigger: "blur"
                    }
                ]
            },
            options: [
                {
                    label: "非国密",
                    value: 0
                },
                {
                    label: "国密",
                    value: 1,
                }
            ]
        }
    },
    mounted: function () {
        // this.chainFrom.type = this.options[0].value
    },
    methods: {
        submit: function (formName) {
            this.$refs[formName].validate(valid => {
                if (valid) {
                    this.loading = true;
                    this.getAllInfo()
                    
                } else {
                    return false
                }
            })
        },
        getAllInfo: function () {
            let type = this.dialogType;
            switch (type) {
                case "creat":
                    this.getCreatChainInfo();
                    break;
                case "modify":
                    this.getModifyChainInfo();
                    break;
            }
        },
        getCreatChainInfo: function () {
            let data = {
                chainName: this.chainFrom.chainName,
                encryptType: this.chainFrom.type,
                description: this.chainFrom.description,
                chainId: this.chainFrom.chainId,
            }
            addChain(data).then(res => {
                this.loading = false
                if (res.data.code === 0) {
                    this.$message({
                        type: 'success',
                        message: '新增成功'
                    })
                    this.modelClose();
                } else {
                    this.$message({
                        type: "error",
                        message: this.$chooseLang(res.data.code)
                    })
                }
            }).catch(err => {
                this.loading = false
                this.$message({
                    type: "error",
                    message: "系统错误"
                })
            })
        },
        getModifyChainInfo: function () {
            let data = {
                chainName: this.chainFrom.chainName,
                // chainType: this.chainFrom.type,
                description: this.chainFrom.description,
                chainId: this.chainFrom.chainId,
            }
            modifyChain(data).then(res => {
                this.loading = false
                if (res.data.code === 0) {
                    this.$message({
                        type: 'success',
                        message: '修改成功'
                    })
                    this.modelClose();
                } else {
                    this.$message({
                        type: "error",
                        message: this.$chooseLang(res.data.code)
                    })
                }
            }).catch(err => {
                this.loading = false
                this.$message({
                    type: "error",
                    message: "系统错误"
                })
            })
        },
        modelClose: function () {
            this.$emit('close')
        }
    }
}
</script>

<style scoped>
    .chain-type >>> .el-input__inner {
        width: 230px;
    }
</style>