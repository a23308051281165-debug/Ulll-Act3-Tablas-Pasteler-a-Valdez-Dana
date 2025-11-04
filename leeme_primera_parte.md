Parte A — Procedimiento (pasos actualizados)

Crear carpeta del proyecto: UIII_Pasteleria_0777

En el Explorador de archivos o terminal: mkdir UIII_Pasteleria_0777

Abrir VS Code sobre la carpeta:

code UIII_Pasteleria_0777 (o abrir VS Code y File → Open Folder → seleccionar UIII_Pasteleria_0777)

Abrir terminal integrado en VS Code:

Ctrl + \ (oTerminal → New Terminal`)

Crear entorno virtual .venv desde la terminal (Windows):

python -m venv .venv

Activar el entorno virtual (PowerShell) en Windows:

.\.venv\Scripts\Activate.ps1

(Si usas cmd: .\.venv\Scripts\activate)

Seleccionar/activar intérprete de Python en VS Code:

Ctrl+Shift+P → Python: Select Interpreter → elegir .\.venv\Scripts\python.exe

Instalar Django:

pip install django

Crear proyecto backend_pasteleria sin duplicar carpeta (usar el “punto” para crear dentro de la carpeta actual):

Desde la carpeta UIII_Pasteleria_0777 (terminal activa):
django-admin startproject backend_pasteleria .

(el . evita crear una subcarpeta adicional)

Ejecutar servidor en el puerto 8036:

python manage.py runserver 8036

Copiar y pegar el link en el navegador:

http://127.0.0.1:8036/

Crear la aplicación app_pasteleria:

python manage.py startapp app_pasteleria

Aquí el models.py actualizado (ver sección siguiente).
12.5. Realizar migraciones:

python manage.py makemigrations

python manage.py migrate

Primero trabajamos con el MODELO: ORDENES (antes Categoría — ahora Ordenes).

En views.py de app_pasteleria crear las funciones (ejemplos: inicio_pasteleria, agregar_orden, actualizar_orden, realizar_actualizacion_orden, borrar_orden) — sin usar forms.py — ejemplo incluido más abajo.

Crear la carpeta templates dentro de app_pasteleria:

app_pasteleria/templates/

En templates crear archivos HTML: base.html, header.html, navbar.html, footer.html, inicio.html (y subcarpetas para las plantillas CRUD).

En base.html agregar Bootstrap (CDN) para CSS y JS.

En navbar.html incluir opciones principales:

“Sistema de Administración Pasteleria”, “Inicio”, “Productos” (submenu: Agregar, Ver, Actualizar, Borrar), “Ordenes” (submenu: Agregar, Ver, Actualizar, Borrar), “Empleados” (submenu: Agregar, Ver, Actualizar, Borrar). Incluir iconos solo en opciones principales.

En footer.html incluir derechos de autor, fecha del sistema y “Creado por Valdez Perez Dana” y mantenerlo fijado al final (sticky footer).

En inicio.html colocar información del sistema y una imagen desde la red (por ejemplo, una imagen representativa de pastelería).

Crear subcarpeta clientes dentro de app_pasteleria/templates si la necesitas (pero ya que cambiamos enfoque, recomendamos una carpeta ordenes y otra productos).

Crear templates CRUD dentro de app_pasteleria/templates/ordenes/ y .../productos/ y .../empleados/, por ejemplo: agregar_orden.html, ver_ordenes.html (tabla con botones ver/editar/borrar), actualizar_orden.html, borrar_orden.html.

No utilizar forms.py (usar acceso directo request.POST).

Crear urls.py en app_pasteleria y registrar rutas para las funciones de views.py (CRUD).

Agregar app_pasteleria en INSTALLED_APPS en backend_pasteleria/settings.py.

Configurar backend_pasteleria/urls.py para enlazar con app_pasteleria.urls.

Registrar los modelos en app_pasteleria/admin.py y volver a realizar migraciones si es necesario.

Por ahora trabajar sólo con “ORDENES” (dejar pendientes SALA y PELÍCULA del ejemplo anterior).

Utilizar colores suaves y modernos en las plantillas (CSS simple).

No validar entrada de datos (según tu requerimiento).

Al inicio crear la estructura completa de carpetas y archivos.

Proyecto totalmente funcional — finalmente ejecutar servidor en el puerto 8036 (python manage.py runserver 8036).

Parte B — models.py para app_pasteleria

Coloca este archivo en app_pasteleria/models.py. He usado tipos Django adecuados y relaciones razonables: Producto, Orden y Empleado. Orden.productos es ManyToMany con Producto. Empleado puede estar asignado a una orden (campo orden_asignada opcional).

# app_pasteleria/models.py
from django.db import models
from django.utils import timezone

# ==========================================
# MODELO: PRODUCTO
# ==========================================
class Producto(models.Model):
    id_producto = models.AutoField(primary_key=True)  # equivalente numeric(5) pk increments
    nombre = models.CharField(max_length=50)
    descripcion = models.CharField(max_length=50, blank=True, null=True)
    # Cógigo original decía "Precio varchar(20)" — aquí usamos Decimal para manejo correcto
    precio = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.BigIntegerField()  # numeric(20)
    unidad_medida = models.CharField(max_length=50, verbose_name="Unidad de medida (Ej: kg)")
    creado_el = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name = "Producto"
        verbose_name_plural = "Productos"

    def __str__(self):
        return self.nombre

# ==========================================
# MODELO: ORDEN
# ==========================================
class Orden(models.Model):
    ID_Orden = models.AutoField(primary_key=True)  # int(10) pk increments unique
    cliente = models.CharField(max_length=50)
    telefono_cliente = models.CharField(max_length=15, blank=True, null=True)
    direccion_entrega = models.CharField(max_length=200, blank=True, null=True)
    # Productos: many-to-many con Producto (se almacena lista de productos en la orden)
    productos = models.ManyToManyField(Producto, related_name="ordenes", blank=True)
    fecha_y_hora = models.DateTimeField(default=timezone.now)
    tipo_pago = models.CharField(max_length=50, blank=True, null=True)  # numeric(50) -> usar cadena o choice si prefieres
    total = models.DecimalField(max_digits=12, decimal_places=2, default=0.00)

    class Meta:
        verbose_name = "Orden"
        verbose_name_plural = "Ordenes"

    def __str__(self):
        return f"Orden {self.ID_Orden} - {self.cliente}"

    def calcular_total(self):
        # cálculo simple: sumar precio * 1 por producto (si quieres cantidades, habría que añadir through table)
        total = sum([p.precio for p in self.productos.all()])
        self.total = total
        return total

# ==========================================
# MODELO: EMPLEADO
# ==========================================
class Empleado(models.Model):
    id_empleado = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=50)
    telefono = models.CharField(max_length=15, blank=True, null=True)
    puesto = models.CharField(max_length=50, blank=True, null=True)
    email = models.EmailField(max_length=254, blank=True, null=True)
    tipo_contrato = models.CharField(max_length=50, blank=True, null=True)
    salario = models.BigIntegerField(blank=True, null=True)  # salarionumeric(50) integer
    activo = models.BooleanField(default=True)
    # Relación con Orden: empleado puede ser asignado a una orden (opcional)
    orden_asignada = models.ForeignKey(Orden, on_delete=models.SET_NULL, null=True, blank=True, related_name="empleados_asignados")

    class Meta:
        verbose_name = "Empleado"
        verbose_name_plural = "Empleados"

    def __str__(self):
        return self.nombre


Notas sobre decisiones de diseño

productos en Orden es ManyToManyField(Producto) — esto permite varias cantidades/varios productos. Si necesitas guardar cantidades por producto usa una tabla intermedia (through) con campo cantidad.

tipo_pago lo dejé como CharField (el spec decía numeric(50) lo cual no tiene sentido práctico); si quieres lo cambio a IntegerField o a choices.

Empleado.orden_asignada es ForeignKey(Orden) opcional — interpretación de tu especificación ID_orden integer pk >* Ordenes.Cliente.

Usé DecimalField para precios y totales (mejor para finanzas).

Parte C — Ejemplo básico views.py para ORDENES (sin forms.py)

Crea en app_pasteleria/views.py (reemplaza/añade según convenga). Estas vistas usan request.POST y plantillas simples.

# app_pasteleria/views.py
from django.shortcuts import render, redirect, get_object_or_404
from .models import Orden, Producto

def inicio_pasteleria(request):
    return render(request, 'inicio.html', {})

# Listar ordenes
def ver_ordenes(request):
    ordenes = Orden.objects.all().order_by('-fecha_y_hora')
    return render(request, 'ordenes/ver_ordenes.html', {'ordenes': ordenes})

# Agregar orden (simple, sin cantidades ni forms.py)
def agregar_orden(request):
    productos = Producto.objects.all()
    if request.method == 'POST':
        cliente = request.POST.get('cliente', '')
        telefono = request.POST.get('telefono_cliente', '')
        direccion = request.POST.get('direccion_entrega', '')
        tipo_pago = request.POST.get('tipo_pago', '')
        productos_ids = request.POST.getlist('productos')  # lista de ids
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

# Mostrar formulario actualizar
def actualizar_orden(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
    productos = Producto.objects.all()
    if request.method == 'POST':
        # delegamos a realizar_actualizacion_orden
        return realizar_actualizacion_orden(request, pk)
    return render(request, 'ordenes/actualizar_orden.html', {'orden': orden, 'productos': productos})

def realizar_actualizacion_orden(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
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

def borrar_orden(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
    if request.method == 'POST':
        orden.delete()
        return redirect('ver_ordenes')
    return render(request, 'ordenes/borrar_orden.html', {'orden': orden})

Parte D — urls.py de la app y enlace en backend_pasteleria/urls.py

Crea app_pasteleria/urls.py:

# app_pasteleria/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_pasteleria, name='inicio_pasteleria'),
    path('ordenes/', views.ver_ordenes, name='ver_ordenes'),
    path('ordenes/agregar/', views.agregar_orden, name='agregar_orden'),
    path('ordenes/actualizar/<int:pk>/', views.actualizar_orden, name='actualizar_orden'),
    path('ordenes/realizar_actualizacion/<int:pk>/', views.realizar_actualizacion_orden, name='realizar_actualizacion_orden'),
    path('ordenes/borrar/<int:pk>/', views.borrar_orden, name='borrar_orden'),
    # rutas para productos y empleados se agregan de forma similar.
]


En backend_pasteleria/urls.py (archivo raíz) añade:

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_pasteleria.urls')),
]

Parte E — Registrar modelos en admin

Editar app_pasteleria/admin.py:

from django.contrib import admin
from .models import Producto, Orden, Empleado

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('id_producto', 'nombre', 'precio', 'stock', 'unidad_medida')
    search_fields = ('nombre',)

@admin.register(Orden)
class OrdenAdmin(admin.ModelAdmin):
    list_display = ('ID_Orden', 'cliente', 'fecha_y_hora', 'total')
    search_fields = ('cliente',)

@admin.register(Empleado)
class EmpleadoAdmin(admin.ModelAdmin):
    list_display = ('id_empleado', 'nombre', 'puesto', 'activo')
    search_fields = ('nombre',)


Luego:

python manage.py makemigrations app_pasteleria

python manage.py migrate

Parte F — Estructura de carpetas recomendada (inicial)
UIII_Pasteleria_0777/
├─ .venv/
├─ backend_pasteleria/
│  ├─ settings.py
│  ├─ urls.py
│  └─ ...
├─ app_pasteleria/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ header.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  ├─ ordenes/
│  │  │  ├─ ver_ordenes.html
│  │  │  ├─ agregar_orden.html
│  │  │  ├─ actualizar_orden.html
│  │  │  └─ borrar_orden.html
│  │  ├─ productos/
│  │  └─ empleados/
│  ├─ admin.py
│  ├─ apps.py
│  ├─ models.py
│  ├─ views.py
│  ├─ urls.py
│  └─ ...
├─ manage.py

Resumen y siguientes pasos sugeridos

Ya tienes los pasos para crear el proyecto y la app, el models.py adaptado a tus tablas, ejemplos de views.py, urls.py y admin.py.

Siguiente (si quieres que haga ahora): crear los templates HTML básicos (base, navbar, inicio, CRUD de ordenes), o añadir la tabla intermedia para cantidades en Ordenes, o convertir tipo_pago en choices, o generar fixtures de ejemplo.

Dime si quieres que cree los archivos HTML (los hago siguiendo tus reglas: colores suaves, Bootstrap CDN y sin validaciones).
