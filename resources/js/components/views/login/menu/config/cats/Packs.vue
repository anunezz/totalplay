<template>
<div class="row">

    <el-dialog
        :close-on-click-modal="true"
        :close-on-press-escape="true"
        :show-close="true"
        :title="titleModal"
        class="modalUser"
        :visible.sync="modalVisible"
        width="70%"
    >
        <span v-if="titleModal === 'Deshabilitar paquete' || titleModal ==='Activar paquete'" v-text="messageModal"></span>

        <span slot="footer" class="dialog-footer" v-if="titleModal === 'Deshabilitar paquete' || titleModal ==='Activar paquete'">
            <div class="btn-group">
                <button type="button" class="btn btn-sm btn-primary" size="mini" @click="modalVisible = false">Cancelar</button>
                <button type="button" class="btn btn-sm btn-danger" v-if="titleModal === 'Deshabilitar paquete'" @click="activePack()"><i class="el-icon-close"></i> Deshabilitar</button>
                <button v-if="titleModal === 'Activar paquete'" type="button" class="btn btn-success btn-sm" @click="activePack()"> <i class="el-icon-check"></i> Activar</button>
            </div>
        </span>
        <new-pack-component
        v-if="titleModal === 'Nuevo paquete' || titleModal === 'Actualizar paquete'"
        :items="pack"
        @responsePack="responsePack" />
        <filter-component v-if="titleModal === 'Búsqueda avanzada'" @responseFormFilters="responseFormFilters" />
    </el-dialog>


    <div class="col-md-12 py-2">
        <div class="row justify-content-md-between flex-wrap">
            <div class="col-md-6">
                <h4>Configuración de paquetes Totalplay.</h4>
            </div>
            <div class="col-md-6">
                <router-link to="/login/configuracion/catalogos">
                    <button class="btn btn-danger btn-sm float-right"> <i class="el-icon-arrow-left"></i> Regresar</button>
                </router-link>
            </div>
        </div>
    </div>
    <div class="col-md-12 py-2">
        <div class="text-left float-left">
            <strong>Total:</strong> {{ pagination.total }}
        </div>
        <div class="btn-group float-right">
            <button class="btn btn-secondary btn-sm" @click="modal(['filter',[]])" > <i class="el-icon-search"></i> Búsqueda avanzada</button>
            <button class="btn btn-primary btn-sm" @click="modal(['new',[]])"> <i class="el-icon-plus"></i> Nuevo paquete</button>
        </div>
    </div>
    <div class="col-md-12">
        <table class="table">
        <thead class="thead-dark">
            <tr>
                <th scope="col">#</th>
                <th scope="col">Paquete</th>
                <th scope="col">Nombre</th>
                <th scope="col">Descripción</th>
                <th scope="col">Residencia</th>
                <th scope="col">Tipo</th>
                <th scope="col">Estatus</th>
                <th scope="col">Acciones</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(item,index) in packs" :key="index" :class="item.namepack.className">
                <th scope="row" v-text="item.idx"></th>
                <td v-text="item.namepack.name"></td>
                <td v-text="item.name"></td>
                <td v-text="item.description"></td>
                <td v-text="item.cat_frontier.name"></td>
                <td v-text="item.cat_triple_doble.name"></td>
                <td v-text="(item.isActive === 1?'Activo':'Desactivado')"></td>
                <td>
                    <div class="btn-group float-center">
                        <button v-if="item.isActive === 1" class="btn btn-danger btn-sm" @click="modal(['active',[item,false]])"> <i class="el-icon-close"></i></button>
                        <button v-else class="btn btn-warning btn-sm" @click="modal(['active',[item,true]])"> <i class="el-icon-check"></i></button>
                        <button class="btn btn-success btn-sm" @click="modal(['edit',item])" > <i class="el-icon-edit"></i></button>
                    </div>
                </td>
            </tr>
        </tbody>
        </table>
    </div>
    <div class="col-md-12">
        <div class="" style="width:100%; padding: 5px 0px;">
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page.sync="pagination.currentPage"
                :page-sizes="[10, 20, 30, 40]"
                :page-size="parseInt(pagination.perPage)"
                layout="sizes, ->, prev, pager, next"
                :total="pagination.total">
            </el-pagination>
        </div>
    </div>
</div>
</template>

<script>
import newPackscomponent from '../../../../fragments/FormNewPack';
import filterComponent from '../../../../fragments/FilterPack';
import {Globals} from "../../../../../../mixins/Globals";
export default {
    mixins:[Globals],
    components:{
        'new-pack-component': newPackscomponent,
        'filter-component': filterComponent
    },
    data(){
        return {
            packs:[],
            modalVisible:false,
            titleModal: null,
            messageModal: null,
            pack:{
                type: false,
                data: {}
            },
            pagination: {
                currentPage: 1,
                total: 0,
                perPage: 10
            },
            from:0,
            to:0,
            filters:{}
        }
    },
    created(){
        this.getCatsPacks();
    },
    methods: {
        responseFormFilters(data){
            this.modalVisible = false;
            this.titleModal = null;
            this.filters = data;
            this.getCatsPacks(this.pagination.currentPage);
        },
        handleSizeChange(sizePerPage) {
            this.pagination.perPage = sizePerPage;
            this.getCatsPacks(this.pagination.currentPage);
        },
        handleCurrentChange(currentPage) {
            this.pagination.currentPage = currentPage;
            this.getCatsPacks(currentPage);
        },
        modal(data){
            $(".el-dialog__title").css({"color":"black"});
            this.messageModa = null;
            switch (data[0]) {
                case 'new':{
                    this.modalVisible = true;
                    this.titleModal = 'Nuevo paquete';
                    this.pack = {
                        type:false,
                        data:[]
                    };
                break;
                }
                case 'edit':{
                    this.modalVisible = true;
                    this.titleModal = 'Actualizar paquete';
                    this.pack = {
                        type: true,
                        data: data[1]
                    };
                break;
                }
                case 'active':{
                    this.modalVisible = true;
                    this.titleModal = ( data[1][1] )? 'Activar paquete':'Deshabilitar paquete';
                    this.messageModal =( data[1][1] )? '¿Estas completamente seguro de activar este paquete?':'¿Estas completamente seguro de deshabilitar este paquete?';
                    this.pack = {
                        type: true,
                        data: data[1]
                    };
                break;
                }
                case 'filter':{
                    this.modalVisible = true;
                    this.titleModal = 'Búsqueda avanzada';
                    this.messageModal = null;
                break;
                }
                default:{
                    this.modalVisible = false;
                    this.titleModal = null;
                    this.pack = {
                        type: false,
                        data: {}
                    };
                    break;
                }
            }
        },
        responsePack(res){
                if(res){
                    this.modalVisible = false;
                    this.titleModal = '';
                    this.messageModal = null;
                    this.getCatsPacks();
                }
                this.$store._modules.root.state.totalplay.loading = false;
        },
        activePack(){
            this.$store._modules.root.state.totalplay.loading = true;
            axios.post('/api/activePack',{
                packid: this.pack.data[0].id,
                active: this.pack.data[1]
            }).then(response => {
                if(response.data.success){
                    this.getCatsPacks();
                    this.modalVisible = false;
                    this.titleModal = '';
                    this.messageModal = null;
                    this.$store._modules.root.state.totalplay.loading = false;
                    let text = (this.pack.data[1])? 'El paquete fue activado correctamente.':
                    'El paquete fue deshabilitado correctamente.';
                    this.$message({
                        message: text,
                        type: 'success'
                    });
                }
            }).catch(error => {
                this.$store._modules.root.state.totalplay.loading = false;
                this.$message({
                    message: 'No se pudo completar la acción.',
                    type: 'warning'
                });
            });
        },
        getCatsPacks(currentPage = 1){
            this.$store._modules.root.state.totalplay.loading = true;
            axios.post('/api/getCatsPacks',{
                filters: this.filters,
                page: currentPage,
                perPage: this.pagination.perPage
            }).then(response => {
                if(response.data.success){
                    this.packs = [];
                    this.packs = this.idx(response.data.lResults.packs.data,response.data.lResults.packs.from);
                    this.pagination.currentPage = response.data.lResults.packs.current_page;
                    this.pagination.perPage = response.data.lResults.packs.per_page;
                    this.pagination.total = response.data.lResults.packs.total;
                    this.from = response.data.lResults.packs.from;
                    this.to = response.data.lResults.packs.to;
                    this.$store._modules.root.state.totalplay.loading = false;
                }
            }).catch(error => {
                this.$message({
                    message: 'No se pudo completar la acción.',
                    type: 'warning'
                });
                this.$store._modules.root.state.totalplay.loading = false;
            });
        }
    }
}
</script>

<style scoped>
    .nolink{
        text-decoration: none;
        color: black;
    }
</style>
