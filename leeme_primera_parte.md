🧩 1️⃣ Crear carpeta del proyecto

    mkdir UIII_Pasteleria_0777
    cd UIII_Pasteleria_0777

🧩 2️⃣ Abrir VS Code sobre la carpeta
    
    code .

🧩 3️⃣ Abrir terminal integrada en VS Code

En el menú superior:
➡️ Terminal → New Terminal

🧩 4️⃣ Crear entorno virtual
    
    python -m venv .venv

🧩 5️⃣ Activar entorno virtual (PowerShell)
    
    .\.venv\Scripts\Activate.ps1
⚠️ Si usas CMD:

    .\.venv\Scripts\activate

🧩 6️⃣ Seleccionar intérprete de Python en VS Code

Presiona:

Ctrl + Shift + P → Python: Select Interpreter → .venv\Scripts\python.exe

🧩 7️⃣ Instalar Django
    
    pip install django

🧩 8️⃣ Crear el proyecto sin duplicar carpeta
    
    django-admin startproject backend_pasteleria .


El punto final (.) evita crear una subcarpeta adicional.

🧩 9️⃣ Crear la aplicación dentro del proyecto
    
    python manage.py startapp app_pasteleria

🧩 🔟 Ejecutar servidor para probar
    
    python manage.py runserver 8036


Copia el enlace en tu navegador:

http://127.0.0.1:8036/

⚙️ CONFIGURACIÓN
🧩 1️⃣1️⃣ Agregar la app en settings.py

Abre backend_pasteleria/settings.py
Busca la sección:

INSTALLED_APPS = [
    ...
]


Agrega dentro de la lista:

    
    'app_pasteleria',

🧩 1️⃣2️⃣ Crear el archivo models.py

Copia este código en:
📁 app_pasteleria/models.py

    from django.db import models
    from django.utils import timezone

# ===============================
# MODELO: PRODUCTO
# ===============================
    class Producto(models.Model):

    id_producto = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=50)
    descripcion = models.CharField(max_length=50, blank=True, null=True)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.BigIntegerField()
    unidad_medida = models.CharField(max_length=50, verbose_name="Unidad de medida (Ej: kg)")
    creado_el = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.nombre


# ===============================
# MODELO: ORDEN
# ===============================
    class Orden(models.Model): 
    ID_Orden = models.AutoField(primary_key=True)
    cliente = models.CharField(max_length=50)
    telefono_cliente = models.CharField(max_length=15, blank=True, null=True)
    direccion_entrega = models.CharField(max_length=200, blank=True, null=True)
    productos = models.ManyToManyField(Producto, related_name="ordenes", blank=True)
    fecha_y_hora = models.DateTimeField(default=timezone.now)
    tipo_pago = models.CharField(max_length=50, blank=True, null=True)
    total = models.DecimalField(max_digits=12, decimal_places=2, default=0.00)
    
    def __str__(self):
        return f"Orden {self.ID_Orden} - {self.cliente}"
        
    def calcular_total(self):
        total = sum([p.precio for p in self.productos.all()])
        self.total = total
        return total


# ===============================
# MODELO: EMPLEADO
# ===============================

    class Empleado(models.Model):
    id_empleado = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=50)
    telefono = models.CharField(max_length=15, blank=True, null=True)
    puesto = models.CharField(max_length=50, blank=True, null=True)
    email = models.EmailField(max_length=254, blank=True, null=True)
    tipo_contrato = models.CharField(max_length=50, blank=True, null=True)
    salario = models.BigIntegerField(blank=True, null=True)
    activo = models.BooleanField(default=True)
    orden_asignada = models.ForeignKey(Orden, on_delete=models.SET_NULL, null=True, blank=True, related_name="empleados_asignados")
    
    def __str__(self):
        return self.nombre

🧩 1️⃣3️⃣ Registrar los modelos en admin.py

Copia esto en:
    
    📁 app_pasteleria/admin.py

    from django.contrib import admin
    from .models import Producto, Orden, Empleado

    @admin.register(Producto)
    class ProductoAdmin(admin.ModelAdmin):
    list_display = ('id_producto', 'nombre', 'precio', 'stock', 'unidad_medida')

    @admin.register(Orden)
    class OrdenAdmin(admin.ModelAdmin):
    list_display = ('ID_Orden', 'cliente', 'fecha_y_hora', 'total')

    @admin.register(Empleado)
    class EmpleadoAdmin(admin.ModelAdmin):
    list_display = ('id_empleado', 'nombre', 'puesto', 'activo')

🧩 1️⃣4️⃣ Crear el archivo urls.py de la app

Copia este código en:
    
    📁 app_pasteleria/urls.py

    from django.urls import path
    from . import views

    urlpatterns = [
    path('', views.inicio_pasteleria, name='inicio_pasteleria'),
    path('ordenes/', views.ver_ordenes, name='ver_ordenes'),
    path('ordenes/agregar/', views.agregar_orden, name='agregar_orden'),
    path('ordenes/actualizar/<int:pk>/', views.actualizar_orden, name='actualizar_orden'),
    path('ordenes/borrar/<int:pk>/', views.borrar_orden, name='borrar_orden'),
    ]

🧩 1️⃣5️⃣ Conectar URLs en el proyecto principal

Abre:
📁 backend_pasteleria/urls.py

Reemplaza su contenido por:

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_pasteleria.urls')),
    ]

🧩 1️⃣6️⃣ Crear views.py

Copia este código en:
📁 app_pasteleria/views.py

    from django.shortcuts import render, redirect, get_object_or_404
    from .models import Orden, Producto

    def inicio_pasteleria(request):
    return render(request, 'inicio.html')

    def ver_ordenes(request):
    ordenes = Orden.objects.all().order_by('-fecha_y_hora')
    return render(request, 'ordenes/ver_ordenes.html', {'ordenes': ordenes})

    def agregar_orden(request):
    productos = Producto.objects.all()
    if request.method == 'POST':
        cliente = request.POST.get('cliente', '')
        telefono = request.POST.get('telefono_cliente', '')
        direccion = request.POST.get('direccion_entrega', '')
        tipo_pago = request.POST.get('tipo_pago', '')
        productos_ids = request.POST.getlist('productos')
        orden = Orden.objects.create(
            cliente=cliente,
            telefono_cliente=telefono,
            direccion_entrega=direccion,
            tipo_pago=tipo_pago
        )
        if productos_ids:
            orden.productos.set(Producto.objects.filter(id__in=productos_ids))
        orden.total = orden.calcular_total()
        orden.save()
        return redirect('ver_ordenes')
    return render(request, 'ordenes/agregar_orden.html', {'productos': productos})

    def actualizar_orden(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
    productos = Producto.objects.all()
    if request.method == 'POST':
        orden.cliente = request.POST.get('cliente', orden.cliente)
        orden.telefono_cliente = request.POST.get('telefono_cliente', orden.telefono_cliente)
        orden.direccion_entrega = request.POST.get('direccion_entrega', orden.direccion_entrega)
        orden.tipo_pago = request.POST.get('tipo_pago', orden.tipo_pago)
        productos_ids = request.POST.getlist('productos')
        if productos_ids:
            orden.productos.set(Producto.objects.filter(id__in=productos_ids))
        orden.total = orden.calcular_total()
        orden.save()
        return redirect('ver_ordenes')
    return render(request, 'ordenes/actualizar_orden.html', {'orden': orden, 'productos': productos})

    def borrar_orden(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
    if request.method == 'POST':
        orden.delete()
        return redirect('ver_ordenes')
    return render(request, 'ordenes/borrar_orden.html', {'orden': orden})

🧩 Crear el proyecto sin duplicar carpeta
🩷 1️⃣ base.html

    <!DOCTYPE html>
    <html lang="es">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block titulo %}Pastelería{% endblock %}</title>
    <!-- Bootstrap -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Estilo personalizado -->
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    </head>
    <body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}

    <main class="container mt-4">
        {% block contenido %}{% endblock %}
    </main>

    {% include 'footer.html' %}

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    </body>
    </html>
    
2️⃣ header.html

    <header class="bg-light text-center py-3 shadow-sm">
        <h1 class="fw-bold text-primary">🎂 Sistema de Administración de Pastelería 🎂</h1>
    </header>
🍰 3️⃣ navbar.html
  
    <nav class="navbar navbar-expand-lg navbar-dark bg-pink shadow">
      <div class="container-fluid">
        <a class="navbar-brand fw-bold" href="{% url 'inicio' %}">Pastelería</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#menuPasteleria">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="menuPasteleria">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">

        <li class="nav-item"><a class="nav-link" href="{% url 'inicio' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">Productos</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_producto' %}">Agregar</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_productos' %}">Ver</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">Órdenes</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar</a></li>
            <li><a class="dropdown-item" href="#">Ver</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">Empleados</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar</a></li>
            <li><a class="dropdown-item" href="#">Ver</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>

🌸 4️⃣ footer.html

    <footer class="bg-light text-center py-3 mt-5 border-top">
    <p class="mb-0">
        &copy; <span id="year"></span> | Creado por <strong>Valdez Pérez Dana</strong> 🍰
    </p>
    </footer>

    <script>
    document.getElementById("year").textContent = new Date().getFullYear();
    </script>

🎀 5️⃣ inicio.html

    {% extends 'base.html' %}
    {% block titulo %}Inicio | Pastelería{% endblock %}

    {% block contenido %}
    <div class="text-center">
    <h2 class="text-primary mb-3">Bienvenido al Sistema de Administración de Pastelería</h2>
    <p class="lead">Gestiona fácilmente tus productos, órdenes y empleados con un solo clic.</p>
    <img src="https://cdn.pixabay.com/photo/2017/08/30/12/45/cake-2690604_1280.jpg" alt="Pastelería" class="img-fluid rounded shadow-lg mt-3" width="60%">
    </div>
    {% endblock %}

🧁 6️⃣ agregar_producto.html
    
    {% extends 'base.html' %}
    {% block titulo %}Agregar Producto{% endblock %}

    {% block contenido %}
    <h2>Agregar Producto</h2>
    <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button class="btn btn-success">Guardar</button>
    <a href="{% url 'ver_productos' %}" class="btn btn-secondary">Cancelar</a>
    </form>
    {% endblock %}

🍩 7️⃣ ver_productos.html
   
    {% extends 'base.html' %}
    {% block titulo %}Lista de Productos{% endblock %}

    {% block contenido %}
    <h2>Lista de Productos</h2>
    <table class="table table-striped">
    <thead class="table-pink">
        <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Precio</th>
            <th>Categoría</th>
            <th>Stock</th>
            <th>Acciones</th>
        </tr>
    </thead>
    <tbody>
        {% for producto in productos %}
        <tr>
            <td>{{ producto.id }}</td>
            <td>{{ producto.nombre }}</td>
            <td>${{ producto.precio }}</td>
            <td>{{ producto.categoria }}</td>
            <td>{{ producto.stock }}</td>
            <td>
                <a href="{% url 'actualizar_producto' producto.id %}" class="btn btn-warning btn-sm">Editar</a>
                <a href="{% url 'borrar_producto' producto.id %}" class="btn btn-danger btn-sm">Borrar</a>
            </td>
        </tr>
        {% empty %}
        <tr>
            <td colspan="6" class="text-center">No hay productos registrados</td>
        </tr>
        {% endfor %}
    </tbody>
    </table>
    {% endblock %}

🧇 8️⃣ actualizar_producto.html

    {% extends 'base.html' %}
    {% block titulo %}Actualizar Producto{% endblock %}

    {% block contenido %}
    <h2>Actualizar Producto</h2>
    <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button class="btn btn-primary">Actualizar</button>
    <a href="{% url 'ver_productos' %}" class="btn btn-secondary">Cancelar</a>
    </form>
    {% endblock %}

🍮 9️⃣ borrar_producto.html

    {% extends 'base.html' %}
    {% block titulo %}Borrar Producto{% endblock %}

    {% block contenido %}
    <h2>Confirmar eliminación</h2>
    <p>¿Seguro que deseas eliminar el producto <strong>{{ producto.nombre }}</strong>?</p>
    <form method="post">
        {% csrf_token %}
        <button class="btn btn-danger">Eliminar</button>
        <a href="{% url 'ver_productos' %}" class="btn btn-secondary">Cancelar</a>
    </form>
    {% endblock %}
🧩 1️⃣7️⃣ Realizar migraciones

    python manage.py makemigrations
    python manage.py migrate
    

🧩 1️⃣8️⃣ Crear superusuario (opcional para admin)

    python manage.py createsuperuser

🧩 1️⃣9️⃣ Ejecutar servidor

    python manage.py runserver 8036


Abrir en navegador:

http://127.0.0.1:8036/

🧩 2️⃣0️⃣ Crear estructura de plantillas

Estructura de carpetas dentro de tu app:


     UIII_Pasteleria_0777/
    └── backend_pasteleria/              # Proyecto principal
    ├── .venv/                       # Entorno virtual de Python
    │   ├── Scripts/
    │   ├── Lib/
    │
    ├── backend_pasteleria/        
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py             
    │   ├── urls.py                
    │   └── wsgi.py
    │
    ├── app_pasteleria/            
    │   ├── __init__.py
    │   ├── admin.py               
    │   ├── models.py               
    │   ├── views.py             
    │   ├── urls.py                  
    │   ├── templates/              
    │   │   ├── base.html          
    │   │   ├── header.html         
    │   │   ├── navbar.html       
    │   │   ├── footer.html        
    │   │   ├── inicio.html          
    │   │   └── clientes/        
    │   └── static/                  
    │       └── css/
    │           └── style.css       
    │
    ├── media/                       
    │
    ├── manage.py                   
    │
    └── requirements.txt             

