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

