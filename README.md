

<!-- README.md или отдельный HTML-файл -->
<div class="background-container">
  <!-- Форма загрузки изображения -->
  <form class="upload-form">
    <input type="file" id="imageInput" accept="image/*" />
  </form>

  <!-- Блок с фоновым изображением -->
  <div id="background" class="background-image"></div>
</div>

<style>
.background-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.background-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  /* По умолчанию можно указать изображение из репозитория */
  background-image: url('https://github.com/RUNO-razrab/FROSTtabletop/raw/main/images/default.jpg'); 
}

.upload-form {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 10;
}
</style>

<script>
  const input = document.getElementById('imageInput');
  const background = document.getElementById('background');

  input.addEventListener('change', (event) => {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        background.style.backgroundImage = `url(${e.target.result})`;
      };
      reader.readAsDataURL(file);
    }
  });
</script>
