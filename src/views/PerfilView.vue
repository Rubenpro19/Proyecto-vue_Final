<template>
  <section id="perfil">
    <div class="contenedor">
      <div class="perfil-info">
        <img :src="currentUser.profileImage || '/src/assets/imagenes/fotos-de-perfil.jpg'" alt="Foto de Perfil">
        <h2><b></b>{{ currentUser.name }}</h2>
        <p><b>Correo: </b>{{ currentUser.email }}</p>
        <p><b>Teléfono: </b>{{ currentUser.telefono }}</p>
        <p><b>Usuario: </b>{{ currentUser.username }}</p>
      </div>

      <div class="cursos-asociados">
        <h2>Cursos Asociados</h2>
        <ul class="cursos-lista">
          <router-link v-for="cursoId in currentUser.cursos" :to="{ name: cursoId }" :key="cursoId" class="curso-item">
            {{ obtenerTituloCurso(cursoId) }}
          </router-link>
        </ul>
      </div>

      <div class="editar-perfil">
        <h2>Editar Perfil</h2>
        <form @submit.prevent="handleSubmit">
          <label for="nombre">Nombre y Apellido:</label>
          <input type="text" id="nombre" name="nombre" v-model="editedName">
          <span v-if="!isNameValid" class="error-message">
          El nombre y apellido deben contener solo letras y permiten espacios.Asegúrese de ingresar al menos un nombre y un apellido, y de que cada palabra tenga al menos 4 caracteres.
          </span>

          <label for="correo">Correo:</label>
          <input type="email" id="correo" name="correo" v-model="editedEmail">
          <span v-if="!isEmailValid" class="error-message">No es un correo válido.</span>

          <label for="telefono">Telefono:</label>
          <input type="text" id="telefono" name="telefono" v-model="editedTelefono" pattern="[0-9]+" autocomplete="tel">
          <span v-if="!isTelefonoValid" class="error-message">
            El teléfono solo puede contener números y debe tener 10 dígitos.
          </span>

          <label for="imagen">Imagen de Perfil:</label>
          <input type="file" id="imagen" name="imagen" accept="image/*" @change="handleImageChange">
          <hr>

          <h3>Cambiar Contraseña</h3>
          <label for="contrasena-actual">Contraseña Actual:</label>
          <input type="password" id="contrasena-actual" name="contrasena-actual" v-model="currentPassword"
            autocomplete="current-password" :class="{ 'error': !isCurrentPasswordValid }">
          <span v-if="!isCurrentPasswordValid" class="error-message">La contraseña actual es incorrecta.</span>

          <label for="contrasena-nueva">Contraseña Nueva:</label>
          <input type="password" id="contrasena-nueva" name="contrasena-nueva" v-model="newPassword"
            autocomplete="new-password" :class="{ 'error': !isNewPasswordValid }">
          <span v-if="!isNewPasswordValid" class="error-message">La nueva contraseña debe tener al menos 8
            caracteres.</span>

          <label for="confirmar-contrasena">Confirmar Contraseña:</label>
          <input type="password" id="confirmar-contrasena" name="confirmar-contrasena" v-model="confirmPassword"
            autocomplete="new-password" :class="{ 'error': !isConfirmPasswordValid }">
          <span v-if="!isConfirmPasswordValid" class="error-message">Las contraseñas no coinciden.</span>

          <input type="submit" value="Guardar Cambios">
        </form>
      </div>
    </div>
  </section>
</template>


<script lang="ts">
import { defineComponent } from 'vue';

interface User {
  name: string;
  email: string;
  telefono: string;
  username: string;
  password: string;
  profileImage: string;
  cursos?: string[]; // Agrega la propiedad cursos como un array de strings
}

const cursos = [
  { id: 'Curso1', titulo: 'Curso de Desarrollo Web', descripcion: 'Aprende a crear sitios web profesionales' },
  { id: 'Curso2', titulo: 'Curso de Marketing Digital', descripcion: 'Domina las estrategias de marketing en línea.' },
  { id: 'Curso3', titulo: 'Curso de Desarrollo Python', descripcion: 'Aprende a programar en uno de los lenguajes más populares.' },
  { id: 'Curso4', titulo: 'Curso de Diseño Gráfico', descripcion: 'Aprende a crear gráficos atractivos, logotipos, imágenes y materiales promocionales' },
  { id: 'Curso5', titulo: 'Aplicaciones Móviles', descripcion: 'Aprenderás a usar tecnologías como React Native, Flutter o desarrollo nativo para iOS y Android.' },
  { id: 'Curso6', titulo: 'Gestión de Proyectos', descripcion: 'Aprender a planificar, ejecutar y controlar proyectos de manera eficiente.' },
];

export default defineComponent({
  data() {
    return {
      currentUser: {} as User,
      selectedImage: null as File | null,
      editedName: '',
      editedEmail: '',
      editedTelefono: '',
      currentPassword: '',
      newPassword: '',
      confirmPassword: '',
      isCurrentPasswordValid: true,
      isNewPasswordValid: true,
      isConfirmPasswordValid: true,
      passwordRegex: /^.{8,}$/,
      alertMessage: '',
      isNameValid: true,
      isEmailValid: true,
      isTelefonoValid: true,
      nameRegex: /^[a-zA-ZÀ-ÿ]+(\s[a-zA-ZÀ-ÿ]{4,}){1,15}$/,
      emailRegex: /^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$/,
      telRegex: /^\d{10}$/,
    };
  },
  mounted() {
    this.loadCurrentUser();
  },
  methods: {
    loadCurrentUser() {
      const currentUserString = localStorage.getItem('currentUser');
      if (currentUserString) {
        this.currentUser = JSON.parse(currentUserString);
        // Inicializar los campos editables con los valores actuales
        this.editedName = this.currentUser.name;
        this.editedEmail = this.currentUser.email;
        this.editedTelefono = String(this.currentUser.telefono);
      }

      const storedProfileImage = localStorage.getItem(`profileImage_${this.currentUser.username}`);
      if (storedProfileImage) {
        this.currentUser.profileImage = storedProfileImage;
      }
    },
    handleImageChange(event: Event) {
      const input = event.target as HTMLInputElement;
      if (input.files && input.files.length > 0) {
        this.selectedImage = input.files[0];
      }
    },
    updateCurrentUser() {
      if (this.isNameValid) {
        this.currentUser.name = this.editedName;
      }

      if (this.isEmailValid) {
        this.currentUser.email = this.editedEmail;
      }

      if (this.isTelefonoValid) {
        this.currentUser.telefono = this.editedTelefono;
      }

      if (this.isCurrentPasswordValid && this.isNewPasswordValid && this.isConfirmPasswordValid) {
        this.currentUser.password = this.newPassword;
      }

      if (this.selectedImage) {
        const imageURL = this.uploadImage(this.selectedImage);
        this.currentUser.profileImage = imageURL;
        localStorage.setItem(`profileImage_${this.currentUser.username}`, imageURL);
      }

      localStorage.setItem('currentUser', JSON.stringify(this.currentUser));

      this.selectedImage = null;
    },
    updateCurrentUserInList() {
      const userListString = localStorage.getItem('userList');
      if (userListString) {
        const userList: User[] = JSON.parse(userListString);
        const index = userList.findIndex(user => user.username === this.currentUser.username);
        if (index !== -1) {
          console.log('Updating user in list:', this.currentUser);
          userList[index] = { ...this.currentUser };
          localStorage.setItem('userList', JSON.stringify(userList));
          console.log('userList updated:', userList);
        } else {
          console.log('User not found in userList:', this.currentUser.username);
          // Si el usuario no se encuentra en la lista, añádelo
          userList.push({ ...this.currentUser });
          localStorage.setItem('userList', JSON.stringify(userList));
          console.log('userList updated with new user:', userList);
        }
      } else {
        console.log('No userList found in localStorage');
      }
    },
    validatePasswords() {
      this.isCurrentPasswordValid = this.currentPassword === this.currentUser.password;
      this.isNewPasswordValid = this.passwordRegex.test(this.newPassword);
      this.isConfirmPasswordValid = this.newPassword === this.confirmPassword;
    },

    validateName() {
      this.isNameValid = this.nameRegex.test(this.editedName)
    },
    validateEmail() {
      this.isEmailValid = this.emailRegex.test(this.editedEmail)
    },
    validateTelefono() {
      this.isTelefonoValid = this.telRegex.test(this.editedTelefono)
    },

    handleSubmit() {
      this.validateName();
      this.validateEmail();
      this.validateTelefono();
      this.validatePasswords();

      // Guardar el estado original de la contraseña
      const originalPassword = this.currentUser.password;

      this.updateCurrentUser();
      this.updateCurrentUserInList();

      // Actualizar los datos solo si todas las validaciones son exitosas
      if (this.isNameValid && this.isEmailValid && this.isTelefonoValid && this.isCurrentPasswordValid && this.isNewPasswordValid && this.isConfirmPasswordValid) {
        this.updateCurrentUser();
        this.updateCurrentUserInList();

        // Mostrar alerta si la contraseña cambió
        if (this.currentUser.password !== originalPassword) {
          this.currentPassword = '';
          this.newPassword = '';
          this.confirmPassword = '';
          alert('Contraseña cambiada con éxito.');
        }
      }
    },
    uploadImage(image: File): string {
      const imageURL = URL.createObjectURL(image);
      return imageURL;
    },
    // Método para obtener el título de un curso por su ID
    obtenerTituloCurso(cursoId: string): string {
      const curso = cursos.find((c) => c.id === cursoId);
      return curso ? curso.titulo : 'Curso no encontrado';
    },

  },
});
</script>

<style scoped>
* {
  box-sizing: border-box;
}

#perfil {
  font-family: 'Roboto', sans-serif;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin: 0;
}

h1 {
  font-size: 4em;
}

h2 {
  font-size: 2em;
}

p {
  font-size: 1.8em;
}

.contenedor {
  max-width: 1400px;
  margin: auto;
}

/* Estilos de la sección de perfil */
#perfil {
  flex: 1;
  background-color: #f9f9f9;
  padding: 40px 0;
}


.perfil-info {
  text-align: center;
}

.perfil-info img {
  border-radius: 50%;
  max-width: 150px;
  border: 4px solid #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.perfil-info h2 {
  margin-top: 10px;
  font-size: 2em;
  color: #333;
}

.perfil-info p {
  font-size: 1.2em;
  color: #555;
}

.editar-perfil {
  max-width: 400px;
}

.editar-perfil form {
  text-align: left;
}

.editar-perfil label {
  display: block;
  margin-bottom: 8px;
  font-size: 1.2em;
  color: #333;
}

.editar-perfil input {
  width: 100%;
  padding: 8px;
  margin-bottom: 16px;
}

.editar-perfil hr {
  margin: 20px 0;
  border: 1px solid #ccc;
}

.editar-perfil h3 {
  font-size: 1.5em;
  color: #333;
  margin-bottom: 10px;
}

.editar-perfil input[type="submit"] {
  background-color: blueviolet;
  color: #fff;
  padding: 10px 20px;
  font-size: 1.2em;
  cursor: pointer;
  border: none;
  border-radius: 5px;
}

.editar-perfil input[type="submit"]:hover {
  background-color: rgb(80, 11, 145);
}

.cursos-asociados {
  margin-top: 20px;
}

.cursos-lista {
  list-style-type: none;
  padding: 0;
}

.curso-item {
  margin-bottom: 10px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  display: block;
  text-decoration: none;
  color: #333;
  transition: background-color 0.3s;
}

.curso-item:hover {
  background-color: #f0f0f0;
}

.error {
  border: 1px solid red;
}

.error-message {
  color: red;
}

@media (max-width: 600px) {
  h1 {
    font-size: 2.5em;
  }

  p {
    font-size: 1.5em;
  }

}
</style>