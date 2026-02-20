# TecNM Cuautla
# Natalia Reyes Baños 24680202 Grupo 3 Semestre 4 20-Feb-2026
# 🎥 Escenario Procedural
### 1. Instalar y abrir Blender
*   **Descarga Blender:** Si no lo tienes, bájalo en [blender.org](https://www.blender.org).
  <img width="1890" height="902" alt="image" src="https://github.com/user-attachments/assets/e8ffecad-3a0d-464b-ae05-38749a1b0e7d" />

*   **Abre el entorno:** Inicia Blender y haz clic en la pestaña **Scripting** en el menú superior.
*   **Nuevo Script:** Haz clic en el botón **+ New** para abrir un editor de texto vacío.
    <img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/8fe9ae97-a89b-45b3-8097-99914321cfe4" />
### 2. El Código
  Copia y pega el siguiente código en el editor de Blender. Lee detenidamente el script, en el se especifica que hace el código.
  
```import bpy
import math

# ===============================
# CREAR MATERIAL
# ===============================
def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes["Principled BSDF"]
    bsdf.inputs['Base Color'].default_value = (*color_rgb, 1.0)
    bsdf.inputs['Roughness'].default_value = 0.10
    return mat

# ===============================
# GENERAR ESCENARIO
# ===============================
def generar_escenario():

    # 1. Limpiar escena
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete(use_global=False)

    # 2. Materiales
    mat_pared_a = crear_material("ParedOscura", (0.3, 0.1, 0.1))
    mat_pared_b = crear_material("ParedDetalle", (0.2, 0.2, 0.0))

    # 3. Parámetros
    largo_pasillo = 10
    ancho_pasillo = 4
    radio_curva = 12

    # ===============================
    # 4A. TRAMO RECTO
    # ===============================
    for i in range(largo_pasillo):

        # Pared izquierda
        bpy.ops.mesh.primitive_cube_add(location=(-ancho_pasillo, i * 2, 1))
        pared_izq = bpy.context.active_object

        if i % 2 == 0:
            pared_izq.data.materials.append(mat_pared_a)
        else:
            pared_izq.data.materials.append(mat_pared_b)
            pared_izq.scale.z = 1.5

        # Pared derecha
        bpy.ops.mesh.primitive_cube_add(location=(ancho_pasillo, i * 2, 1))
        pared_der = bpy.context.active_object
        pared_der.data.materials.append(mat_pared_a)

    # ===============================
    # 4B. TRAMO CURVO
    # ===============================
    cy = (largo_pasillo - 1) * 2
    cx = radio_curva

    for j in range(1, largo_pasillo + 1):
        angulo = math.pi - (j * (math.pi / 2) / largo_pasillo)
        rotacion_z = math.pi - angulo

        # Pared izquierda curva
        x_izq = cx + (radio_curva + ancho_pasillo) * math.cos(angulo)
        y_izq = cy + (radio_curva + ancho_pasillo) * math.sin(angulo)

        bpy.ops.mesh.primitive_cube_add(
            location=(x_izq, y_izq, 1),
            rotation=(0, 0, rotacion_z)
        )
        pared_izq_curva = bpy.context.active_object

        if j % 2 == 0:
            pared_izq_curva.data.materials.append(mat_pared_a)
        else:
            pared_izq_curva.data.materials.append(mat_pared_b)
            pared_izq_curva.scale.z = 1.5

        # Pared derecha curva
        x_der = cx + (radio_curva - ancho_pasillo) * math.cos(angulo)
        y_der = cy + (radio_curva - ancho_pasillo) * math.sin(angulo)

        bpy.ops.mesh.primitive_cube_add(
            location=(x_der, y_der, 1),
            rotation=(0, 0, rotacion_z)
        )
        pared_der_curva = bpy.context.active_object
        pared_der_curva.data.materials.append(mat_pared_a)

    # ===============================
    # 5. SUELO
    # ===============================
    bpy.ops.mesh.primitive_plane_add(
        size=1,
        location=(radio_curva/2, cy/2 + radio_curva/2, 0)
    )
    suelo = bpy.context.active_object
    suelo.scale.x = (ancho_pasillo * 2) + radio_curva + 10
    suelo.scale.y = (largo_pasillo * 2) + radio_curva + 10

    # ===============================
    # 6. LUCES
    # ===============================
    bpy.ops.object.light_add(
        type='SUN',
        location=(10, 10, 10),
        rotation=(math.radians(-45), math.radians(30), 0)
    )
    bpy.context.active_object.data.energy = 3

    bpy.ops.object.light_add(type='POINT', location=(0, 5, 4))
    bpy.context.active_object.data.energy = 500

    bpy.ops.object.light_add(
        type='POINT',
        location=(radio_curva, cy + radio_curva/2, 4)
    )
    bpy.context.active_object.data.energy = 800

    # ===============================
    # 7. CÁMARA
    # ===============================
    bpy.ops.object.camera_add(location=(0, 0, 0))
    camera = bpy.context.active_object
    bpy.context.scene.camera = camera

    # ===============================
    # 8. CAMINO DE CÁMARA
    # ===============================
    curve_data = bpy.data.curves.new('CamPathData', type='CURVE')
    curve_data.dimensions = '3D'
    curve_data.use_path = True

    spline = curve_data.splines.new('POLY')

    puntos_camino = [(0, -6, 1.5), (0, cy, 1.5)]
    pasos_curva = 20

    for frame in range(1, pasos_curva + 1):
        progreso = frame / float(pasos_curva)
        angulo = math.pi - (progreso * (math.pi / 2))
        px = cx + radio_curva * math.cos(angulo)
        py = cy + radio_curva * math.sin(angulo)
        puntos_camino.append((px, py, 1.5))

    spline.points.add(len(puntos_camino) - 1)

    for i, pt in enumerate(puntos_camino):
        spline.points[i].co = (*pt, 1.0)

    cam_path = bpy.data.objects.new('Cam_Path', curve_data)
    bpy.context.scene.collection.objects.link(cam_path)

    # ===============================
    # 9. TARGET
    # ===============================
    bpy.ops.object.empty_add(type='PLAIN_AXES', location=(0, 0, 0))
    cam_target = bpy.context.active_object
    cam_target.name = "Cam_Target"

    fp_target = cam_target.constraints.new(type='FOLLOW_PATH')
    fp_target.target = cam_path
    fp_target.use_fixed_location = True

    # Cámara constraints
    follow_path = camera.constraints.new(type='FOLLOW_PATH')
    follow_path.target = cam_path
    follow_path.use_fixed_location = True

    track_to = camera.constraints.new(type='TRACK_TO')
    track_to.target = cam_target
    track_to.track_axis = 'TRACK_NEGATIVE_Z'
    track_to.up_axis = 'UP_Y'

    # ===============================
    # 10. ANIMACIÓN
    # ===============================
    bpy.context.scene.frame_start = 1
    bpy.context.scene.frame_end = 200

    follow_path.offset_factor = 0.0
    fp_target.offset_factor = 0.05

    camera.keyframe_insert(
        data_path=f'constraints["{follow_path.name}"].offset_factor',
        frame=1
    )
    cam_target.keyframe_insert(
        data_path=f'constraints["{fp_target.name}"].offset_factor',
        frame=1
    )

    follow_path.offset_factor = 0.95
    fp_target.offset_factor = 1.0

    camera.keyframe_insert(
        data_path=f'constraints["{follow_path.name}"].offset_factor',
        frame=200
    )
    cam_target.keyframe_insert(
        data_path=f'constraints["{fp_target.name}"].offset_factor',
        frame=200
    )

    bpy.context.view_layer.update()


# ===============================
# EJECUTAR 
# ===============================
generar_escenario()
```

### 3. Resultado final
Por último el escenario procedural debió de quedar de la siguiente manera:
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/afaa2e89-e676-48f9-aa67-ee08de182ade" />

### 4. 🎥 Visualizar desde la cámara
Ve al ícono de una cámara y haz click
<img width="121" height="243" alt="image" src="https://github.com/user-attachments/assets/f7ba7d24-fb46-407e-be64-44e35c752faa" />

Te llevará aquí
<img width="1156" height="906" alt="image" src="https://github.com/user-attachments/assets/70eacdc1-5d1a-47a9-922b-5ac4600ea004" />

Y para finalizar da click en la barra de espacio para dar play a la animación!!!
