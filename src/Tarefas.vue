<template>
  <a class="fixo button is-large is-danger is-loading" v-show="isLoading">Loading</a>
  <div class="container">
    <h1 class="title">{{title}}</h1>
    <div class="columns">
      <div class="column is-5">
        <p class="control has-addons">
          <input class="input is-expanded" type="text" placeholder="Procure pelo nome ou descrição" v-model="search">
          <a class="button is-info" @click.prevent="searchTarefas">Pesquisar</a>
        </p>
      </div>
      <div class="column is-6">
         
      </div>
      <div class="column is-1">
        <a class="button is-info" @click.prevent="newTarefa">Novo</a>
      </div>

    </div>
    <div class="columns">
      <div class="column is-12">
        <table class="table is-narrow is-bordered">
          <thead>
            <th>Nome</th>
			<th>Descrição</th>
			<th>Última Modificação</th>
			<th>Data/Hora Inserção</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tarefa in tarefas">
            <td>{{tarefa.name}}</td>
			<td>{{tarefa.descript}}</td>
			<td>{{moment(tarefa.last_mod).format('MMMM Do YYYY, h:mm:ss a')}}</td>
			<td>{{moment(tarefa.data_hora_insercao).format('MMMM Do YYYY, h:mm:ss a')}}</td>
            <td class="is-icon">

              <a href="#" @click.prevent="editTarefa(tarefa)">
                <i class="fa fa-edit"></i>
              </a>
              <a href="#" @click.prevent="removeTarefa(tarefa)">
                <i class="fa fa-trash"></i>
              </a>
            </td>
          </tr>
        </tbody>
      </table>
      <Pagination :total="total" :page="page" :itens-per-page="itensPerPage" @change-page="onChangePage"></Pagination>
    </div>
  </div>
</div>

<div id="modal_tarefa" class="modal" :class="{'is-active':showModal}">
  <div class="modal-background"></div>
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Tarefa: {{selected.name}}</p>
      <button class="delete" @click.prevent="showModal=false"></button>
    </header>
    <section class="modal-card-body">

    <div class="columns">
      <div class="column">
        <label class="label">Nome</label>
          <p class="control">
            <input class="input" type="text" placeholder="Text input" v-model="selected.name">
          </p>
      </div>
      
      </div>

      <label class="label">Descrição</label>
      <p class="control">
        <textarea class="textarea" placeholder="Textarea" v-model="selected.descript"></textarea>
      </p>
      
      <label class="label">Data e Hora</label>
      <p class="control">
        <textarea class="input" placeholder="Text input" v-model="selected.data_hora_insercao"></textarea>
      </p>	  
	  
	<label class="label">Última Modificação</label>
    <p class="control">
      <input class="input" type="text" placeholder="Text input" v-model="selected.last_mod">
    </p>
	
    </section>
    <footer class="modal-card-foot">
      <a class="button is-primary" @click.prevent="saveTarefa">Salvar</a>
      <a class="button" @click.prevent="showModal=false">Cancelar</a>
    </footer>
  </div>
</div>
</template>

<script>
  import Pagination from './Pagination.vue'
  

  export default {
    data () {
      return {
        isLoading: false,
        title: 'Lista de Tarefas',
        search: '',
        tarefas: [],
        page: 1,
        total: 0,
        selected: {},
        itensPerPage: 10,
        showModal:false
      }
    },
    components: {
      Pagination
    },
    methods: {
      onChangePage(page){
        this.page = page
        this.loadTarefas()
      },
      showLoading(){
        this.isLoading=true;
      },
      hideLoading(){
        this.isLoading=false;
      },
      loadTarefas(){

        let t = this
        this.showLoading()

        let start = (this.page * this.itensPerPage) - (this.itensPerPage)
        let end = this.page * this.itensPerPage
        let qString = "";

        if (this.search){
          qString = `&q=${this.search}`
        }

        this.$http.get(`/tarefas?_start=${start}&_end=${end}${qString}`).then(
         response=>{
           t.tarefas = response.json()
           t.total = response.headers['X-Total-Count']
         },
         error=>{
           console.log(error)
         }).finally(function () {
          t.hideLoading();
        })

       },
       searchTarefas(){
        this.loadTarefas()
       },
       newTarefa(){
        this.selected={}
        this.showModal = true;
       },
       editTarefa(tarefa){
        this.selected=tarefa
        this.showModal = true;
       },
       removeTarefa(tarefa){
        let self = this;
        swal({  title: "Você tem certeza?",
                 text: `Deseja apagar "${tarefa.name}"`,   
                 type: "warning",   
                 showCancelButton: true,   
                 confirmButtonColor: "#DD6B55",   
                 cancelButtonText: "Cancelar",
                 confirmButtonText: "Sim, pode apagar!", 
                 showLoaderOnConfirm: true,  
                 closeOnConfirm: false }, function(){   
                  
                  self.$http.delete(`/tarefas/${tarefa.id}`).then(
                    result=>{
                      swal("Tarefa removida!")
                      self.loadTarefas()
                    })
        });

       },
       saveTarefa(){
        if (this.selected.id!=null){  //EDIT
          this.$http.put(`/tarefas/${this.selected.id}`,this.selected).then(
            response=>{
              this.$set('selected',{})
              this.$set('showModal',false)
            },
            error=>{
              console.error(error)
            }
            ).finally(
              this.loadTarefas()
            )
          }
          else
          { //NEW
            this.$http.post(`/tarefas`,this.selected).then(
            response=>{
              this.$set('selected',{})
              this.$set('showModal',false)
            },
            error=>{
              console.error(error)
            }
            ).finally(
              this.loadTarefas()
            )
          }
       },
	   moment: function (date) {
		  return moment(date);
		},
		date: function (date) {
		  return moment(date).format('MMMM Do YYYY, h:mm:ss a');
		}
     },
     created(){
      this.loadTarefas();
    },
	filters: {
		moment: function (date) {
		  return moment(date).format('MMMM Do YYYY, h:mm:ss a');
		}
	  },
  }
  
</script>
<style>
  .fixo{float: right; margin-right: 10px; margin-top: 0px; z-index: 1000;}
  
  .method {  color: grey; }

  .filter {  color:green;}
  
</style>
