<template>
<main class="main">
	<!-- Breadcrumb -->
	<ol class="breadcrumb">
		<li class="breadcrumb-item"><a href="/">Escritorio</a></li>
	</ol>
	<div class="container-fluid">
		<!-- Ejemplo de tabla Listado -->
		<div class="card">
			<div class="card-header">
				<i class="fa fa-align-justify"></i> Categorías
				<button type="button" class="btn btn-secondary" @click="abrirModal('categoria', 'registrar')">
          <i class="icon-plus"></i>&nbsp;Nuevo
        </button>
			</div>
			<div class="card-body">
				<div class="form-group row">
					<div class="col-md-6">
						<div class="input-group">
							<select class="form-control col-md-3" v-model="criterio">
                <option value="nombre">Nombre</option>
                <option value="descripcion">Descripción</option>
              </select>
							<input type="text" v-model="buscar" @keyup.enter="listarCategoria(1, buscar, criterio)" class="form-control" placeholder="Texto a buscar">
							<button type="submit" class="btn btn-primary" @click.prevent="listarCategoria(1, buscar, criterio)"><i class="fa fa-search"></i> Buscar</button>
						</div>
					</div>
				</div>
				<table class="table table-bordered table-striped table-sm">
					<thead>
						<tr>
							<th>Opciones</th>
							<th>Nombre</th>
							<th>Descripción</th>
							<th>Estado</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="categoria in arrayCategoria" :key="categoria.id">
							<td>
								<button type="button" class="btn btn-warning btn-sm" @click="abrirModal('categoria', 'actualizar', categoria)">
                  <i class="icon-pencil"></i>
                </button> &nbsp;
                <template v-if="categoria.condicion">
                  <button type="button" class="btn btn-danger btn-sm" @click="desactivarCategoria(categoria.id)">
                    <i class="icon-trash"></i>
                  </button>
                </template>
                <template v-else>
                  <button type="button" class="btn btn-info btn-sm" @click="activarCategoria(categoria.id)">
                    <i class="icon-check"></i>
                  </button>
                </template>
							</td>
							<td v-text="categoria.nombre"></td>
							<td v-text="categoria.descripcion"></td>
							<td>
                <span :class="[categoria.condicion == true ? 'badge-success' : 'badge-danger']" class="badge">{{ ((categoria.condicion == true) ? 'Activo' : 'Desactivado') }}</span>
							</td>
						</tr>
					</tbody>
				</table>
				<nav>
					<ul class="pagination">
						<li class="page-item" v-if="pagination.current_page > 1">
							<a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page - 1, buscar, criterio)">Ant</a>
						</li>
						<li class="page-item" v-for="page in pagesNumber" :key="page" :class="[page == isActived ? 'active' : '']">
							<a class="page-link" href="#" @click.prevent="cambiarPagina(page, buscar, criterio)" v-text="page"></a>
						</li>
						<li class="page-item" v-if="pagination.current_page < pagination.last_page">
							<a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page + 1, buscar, criterio)">Sig</a>
						</li>
					</ul>
				</nav>
			</div>
		</div>
		<!-- Fin ejemplo de tabla Listado -->
	</div>
	<!--Inicio del modal agregar/actualizar-->
	<div class="modal fade" tabindex="-1" :class="{'mostrar': modal}" role="dialog" aria-labelledby="myModalLabel" style="display: none;" aria-hidden="true">
		<div class="modal-dialog modal-primary modal-lg" role="document">
			<div class="modal-content">
				<div class="modal-header">
					<h4 class="modal-title">{{ tituloModal }}</h4>
					<button type="button" class="close" @click="cerrarModal" aria-label="Close">
            <span aria-hidden="true">×</span>
          </button>
				</div>
				<div class="modal-body">
					<form action="" method="post" enctype="multipart/form-data" class="form-horizontal">
						<div class="form-group row">
							<label class="col-md-3 form-control-label" for="text-input">Nombre</label>
							<div class="col-md-9">
								<input type="text" v-model="nombre" class="form-control" placeholder="Nombre de categoría">
							</div>
						</div>
						<div class="form-group row">
							<label class="col-md-3 form-control-label">Descripción</label>
							<div class="col-md-9">
								<input type="text" v-model="descripcion" class="form-control" placeholder="Ingrese Descripción">
							</div>
						</div>
            <div v-show="errorCategoria" class="form-group row">
              <div class="col-12 text-center">
                <div class="alert alert-danger">
                  <p class="mb-0" v-for="error in errorsMsgs" v-text="error"></p>
                </div>
              </div>
            </div>
					</form>
				</div>
				<div class="modal-footer">
					<button type="button" class="btn btn-secondary" @click="cerrarModal">Cerrar</button>
					<button type="button" class="btn btn-primary" v-if="tipoAccion==1" @click="registrarCategoria">Guardar</button>
          <button type="button" class="btn btn-primary" v-else-if="tipoAccion==2" @click="actualizarCategoria">Actualizar</button>
				</div>
			</div>
			<!-- /.modal-content -->
		</div>
		<!-- /.modal-dialog -->
	</div>
	<!--Fin del modal-->
</main>
</template>

<script>
export default {
  data() {
    return {
      categoria_id: 0,
      nombre: '',
      descripcion: '',
      arrayCategoria: [],
      modal: 0,
      tituloModal: '',
      tipoAccion: 0,
      errorCategoria: 0,
      errorsMsgs: [],
      pagination: {
        total: 0,
        current_page: 0,
        per_page: 0,
        last_page: 0,
        from: 0,
        to: 0
      },
      offset: 3,
      criterio: 'nombre',
      buscar: ''
    };
  },
  computed: {
    isActived(){
      return this.pagination.current_page;
    },
    pagesNumber(){
      if (!this.pagination.to) {
        return [];
      }

      let from = this.pagination.current_page - this.offset;

      if (from < 1) {
        from = 1;
      }

      let to = from + (this.offset * 2);

      if (to >= this.pagination.last_page) {
        to = this.pagination.last_page;
      }

      let pagesArray = [];
      while (from <= to) {
        pagesArray.push(from);
        from++;
      }

      return pagesArray;
    }
  },
  methods: {
    listarCategoria(page, buscar, criterio){
      let self = this;
      let url = '/categoria?page=' + page + '&buscar=' + buscar + '&criterio=' + criterio;

      axios.get(url)
        .then(function (response) {
          let respuesta = response.data;
          self.arrayCategoria = respuesta.categorias.data;
          self.pagination = respuesta.pagination;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    cambiarPagina(page, buscar, criterio){
      let self = this;
      self.pagination.current_page = page;
      self.listarCategoria(page, buscar, criterio);
    },
    registrarCategoria(){
      if (this.validarCategoria()) {
        return;
      }
      let self = this;
      axios.post('/categoria/registrar', {
        nombre: self.nombre,
        descripcion: self.descripcion
      })
      .then(function (response) {
        self.cerrarModal();
        self.listarCategoria(1, '', 'nombre');
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    actualizarCategoria(){
      if (this.validarCategoria()) {
        return;
      }
      let self = this;
      axios.put('/categoria/actualizar', {
        id: self.categoria_id,
        nombre: self.nombre,
        descripcion: self.descripcion
      })
      .then(function (response) {
        self.cerrarModal();
        self.listarCategoria(1, '', 'nombre');
      })
      .catch(function (error) {
        console.log(error);
      });
    },
    desactivarCategoria(id){
      let self = this;
      const swalWithBootstrapButtons = swal.mixin({
        confirmButtonClass: 'btn btn-success',
        cancelButtonClass: 'btn btn-danger',
        buttonsStyling: false,
      });

      swalWithBootstrapButtons({
        title: '¿Estás seguro de desactivar esta categoría?',
        type: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Desactivar',
        cancelButtonText: 'Cancelar',
        reverseButtons: true
      }).then((result) => {
        if (result.value) {
          axios.put('/categoria/desactivar', {
            id: id
          })
          .then(function (response) {
            swalWithBootstrapButtons(
              'Desactivado',
              'El registro ha sido desactivado con éxito.',
              'success'
            );
            self.listarCategoria(1, '', 'nombre');
          })
          .catch(function (error) {
            console.log(error);
          });
        } else if (
          // Read more about handling dismissals
          result.dismiss === swal.DismissReason.cancel
        ) {
          //nothing to do
        }
      });
    },
    activarCategoria(id){
      let self = this;
      const swalWithBootstrapButtons = swal.mixin({
        confirmButtonClass: 'btn btn-success',
        cancelButtonClass: 'btn btn-danger',
        buttonsStyling: false,
      });

      swalWithBootstrapButtons({
        title: '¿Estás seguro de Activar esta categoría?',
        type: 'warning',
        showCancelButton: true,
        confirmButtonText: 'Activar',
        cancelButtonText: 'Cancelar',
        reverseButtons: true
      }).then((result) => {
        if (result.value) {
          axios.put('/categoria/activar', {
            id: id
          })
          .then(function (response) {
            swalWithBootstrapButtons(
              'Activado',
              'El registro ha sido activado con éxito.',
              'success'
            );
            self.listarCategoria(1, '', 'nombre');
          })
          .catch(function (error) {
            console.log(error);
          });
        } else if (
          // Read more about handling dismissals
          result.dismiss === swal.DismissReason.cancel
        ) {
          //nothing to do
        }
      });
    },
    validarCategoria(){
      this.errorCategoria = 0;
      this.errorsMsgs = [];
      if (!this.nombre) {
        this.errorsMsgs.push('El nombre de la categoría no puede estar vacia.');
      }
      if (this.errorsMsgs.length) {
        this.errorCategoria = 1;
      }
      return this.errorCategoria;
    },
    abrirModal(modelo, accion, data = []){
      switch (modelo) {
        case 'categoria':
          switch (accion) {
            case 'registrar':
              this.modal = 1;
              this.tipoAccion = 1;
              this.nombre = '';
              this.descripcion = '';
              this.tituloModal = 'Registrar Categoría';
              break;
            case 'actualizar':
              this.modal = 1;
              this.tipoAccion = 2;
              this.categoria_id = data.id;
              this.nombre = data.nombre;
              this.descripcion = data.descripcion;
              this.tituloModal = 'Actualizar Categoría';
              break;
          }
          break;
      }
    },
    cerrarModal(){
      this.modal = 0;
      this.tituloModal = '';
      this.nombre = '';
      this.descripcion = '';
      this.errorCategoria = 0;
    }
  },
	mounted() {
    this.listarCategoria(1, this.buscar, this.criterio);
	}
}
</script>

<style>
  .modal-content{
    width: 100% !important;
    position: absolute !important;
  }
  .mostrar{
    display: list-item !important;
    opacity: 1 !important;
    position: absolute !important;
    background-color: #3C29297A !important;
  }
</style>
