bl_info = {
    "name": "MatricX",
    "author": "Jack",
    "version": (1, 5, 0),
    "blender": (4, 4, 3),
    "location": "View3D > Sidebar > MatricX",
    "description": "Generate smart materials with color, type,",
    "category": "Material",
}

import bpy
import os

preview_collections = {}

__version__ = (1, 5, 0)  # major

import webbrowser

class OT_CheckForUpdate(bpy.types.Operator):
    bl_idname = "matricx.check_update"
    bl_label = "Check for Update"
    bl_description = "Opens the update page in your browser"

    def execute(self, context):
        # 🔁 Replace with your actual update page (GitHub / Gumroad)
        webbrowser.open("https://jackhardsurface.gumroad.com/l/Matricx")
        self.report({'INFO'}, "Opened update page in browser.")
        return {'FINISHED'}


class MaterialGeneratorProperties(bpy.types.PropertyGroup):
    
    show_help: bpy.props.BoolProperty(
    name="Show Help & Updates",
    description="Show help info and update checker",
    default=False
    )


    brushed_type: bpy.props.EnumProperty(
    name="Brushed Type",
    description="Controls the direction of brushed texture",
    items=[
        ("TYPE1", "Type 1", ""),
    ],
    default="TYPE1",
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

  
    car_paint_color: bpy.props.FloatVectorProperty(
    name="Paint Color",
    subtype='COLOR',
    default=(0.1, 0.2, 0.8),  # A cool blue metallic by default
    min=0.0,
    max=1.0,
    description="Base color of the car paint",
)

    camo_type: bpy.props.EnumProperty(
    name="Camo Type",
    description="Choose camo color palette",
    items=[
        ("TYPE1", "Type 1", "Default green-based camo"),
        ("TYPE2", "Type 2", "Camo Triad"),
        ("TYPE3", "Type 3", "Urban camo"),
        ("TYPE4", "Type 4", "Olive-brown Saratoga scheme"),
        # Add more as you go
    ],
    default="TYPE1",
)


    pattern_randomness: bpy.props.FloatProperty(
    name="Pattern Randomness",
    description="Adjusts the randomness of the camo pattern (Voronoi W value)",
    default=3.7,
    min=0.0,
    max=10.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    camo_scale: bpy.props.FloatProperty(
    name="Scale",
    description="Controls the overall scale of the camo pattern",
    default=1.0,
    min=0.0,
    max=10.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)



    edge_wear_color: bpy.props.FloatVectorProperty(
    name="Edge Wear Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(0.5, 0.5, 0.5, 1.0),
       
    )
    
    edge_wear_spread: bpy.props.FloatProperty(
    name="Edge Wear Spread",
    description="Controls the edge wear mask spread",
    default=0.05,  # or whatever value you like
    min=0.0,
    max=0.5,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    edge_wear_strength: bpy.props.FloatProperty(
    name="Edge Wear Strength",
    description="Adjusts edge wear mask intensity (affects Map Range From Min)",
    default=-0.5,
    min=-1.0,
    max=0.9,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    # Down Dirt Properties
    dirt_color_1: bpy.props.FloatVectorProperty(
    name="Dirt Color 1",
    subtype='COLOR',
    default=(0.133, 0.043, 0.0),
    min=0.0,
    max=1.0,
    description="Dark dirt tone",
   
)

    dirt_color_2: bpy.props.FloatVectorProperty(
    name="Dirt Color 2",
    subtype='COLOR',
    default=(0.792, 0.408, 0.168),
    min=0.0,
    max=1.0,
    description="Bright dirt tone",
    
)

    dirt_roughness: bpy.props.FloatProperty(
    name="Dirt Roughness",
    default=0.75,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    dirt_metallic: bpy.props.FloatProperty(
    name="Dirt Metallic",
    default=0.0,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    dirt_opacity: bpy.props.FloatProperty(
    name="Dirt Opacity",
    description="Blends the dirt with the base material (lower = stronger dirt)",
    default=0.0,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    dirt_spread: bpy.props.FloatProperty(
    name="Dirt Spread",
    description="Controls how high the dirt spreads up the mesh",
    default=-1.5,
    min=-2.5,
    max=-0.5,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    dirt_strength: bpy.props.FloatProperty(
    name="Dirt Strength",
    description="Controls the bump strength of down dirt",
    default=0.2,
    min=0.0,
    max=0.5,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    shady_color_1: bpy.props.FloatVectorProperty(
    name="Color 1",
    subtype='COLOR',
    size=4,
    default=(0.0, 0.0, 1.0, 1.0),  # Blue
    min=0.0,
    max=1.0,
    description="First color in the Shady ramp",
    
)

    shady_color_2: bpy.props.FloatVectorProperty(
    name="Color 2",
    subtype='COLOR',
    size=4,
    default=(1.0, 1.0, 1.0, 1.0),  # White
    min=0.0,
    max=1.0,
    description="Second color in the Shady ramp",
    
)

    shady_color_3: bpy.props.FloatVectorProperty(
    name="Color 3",
    subtype='COLOR',
    size=4,
    default=(0.0, 0.0, 0.3, 1.0),  # Dark Blue
    min=0.0,
    max=1.0,
    description="Third color in the Shady ramp",
    
)
    shady_scale: bpy.props.FloatProperty(
    name="Shady Scale",
    description="Controls the scale of the brick distortion",
    default=0.75,
    min=0.1,
    max=3.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)


    rust_spread: bpy.props.FloatProperty(
    name="Rust Spread",
    description="Controls the rust mask spread",
    default=0.05,
    min=0.0,
    max=0.6,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    crack_detailing: bpy.props.FloatProperty(
    name="Crack Detailing",
    description="Adjust the detail level of concrete cracks",
    default=0.7,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    crack_strength: bpy.props.FloatProperty(
    name="Crack Strength",
    description="Adjust the bump intensity of concrete cracks",
    default=0.4,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    concrete_crack_scale: bpy.props.FloatProperty(
    name="Crack Scale",
    description="Uniformly scales the crack and concrete pattern",
    default=1.0,
    min=0.0,
    max=10.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    scratch_color: bpy.props.FloatVectorProperty(
    name="Scratch Color",
    subtype='COLOR',
    size=4,
    default=(0.8, 0.8, 0.8, 1.0),  # Light gray scratches
    min=0.0,
    max=1.0,
    description="Color of the scratch layer",
)
    scratch_size: bpy.props.FloatProperty(
    name="Scratch Size",
    description="Controls the scale of scratch mapping (uniform XYZ)",
    default=1.0,
    min=0.0,
    max=5.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    scratch_location: bpy.props.FloatVectorProperty(
    name="Scratch Location",
    subtype='XYZ',
    default=(0.0, 0.0, 0.0),
    soft_min=0.0,
    soft_max=5.0,
    description="Adjust scratch mapping location on X, Y, Z axes",
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)


    oxidation_mapping_location: bpy.props.FloatVectorProperty(
    name="Oxidation Location",
    subtype='XYZ',
    default=(0.0, 0.0, 0.0),
    soft_min=-5.0,
    soft_max=5.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    oxidation_mapping_scale: bpy.props.FloatProperty(
    name="Oxidation Scale",
    default=0.5,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)


    # ... rest of your properties
    dirt_color: bpy.props.FloatVectorProperty(
    name="Dirt Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(0.5, 0.5, 0.5, 1.0)  # Default grey dirt
)
    # Live Dirt Mapping Location
    dirt_mapping_location: bpy.props.FloatVectorProperty(
    name="Dirt Location",
    subtype='XYZ',
    default=(0.0, 0.0, 0.0),
    soft_min=-5.0,
    soft_max=5.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    gold_roughness: bpy.props.FloatProperty(
    name="Gold Roughness",
    description="Adjust gold roughness (affects second ColorRamp slider)",
    default= 2,
    min=-1.0,
    max=1.0,
    )
    
    
    gold_type: bpy.props.EnumProperty(
    name="Gold Type",
    description="Choose the gold material type",
    items=[
        ("TYPE1", "Procedural", "Standard procedural gold material"),
        ("TYPE2", "Damaged Gold", "Gold with bump and roughness detail"),
    ],
    default="TYPE1"
)

    
    dirt_mapping_scale: bpy.props.FloatProperty(
    name="Dirt Scale",
    default=0.5,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    oxidation_color: bpy.props.FloatVectorProperty(
    name="Oxidation Color",
    subtype='COLOR',
    default=(0.05, 0.05, 0.05),
    min=0.0,
    max=1.0
    # 🔻 Removed update=...
)

    # Dirt Roughness
    dirt_roughness: bpy.props.FloatProperty(
    name="Dirt Roughness",
    description="Controls dirt material roughness",
    default=0.8,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    use_unique_material: bpy.props.BoolProperty(
    name="Unique Material per Object",
    default=False,
    description="Generate and assign a unique material to each object",
)

    rust_darkness: bpy.props.FloatProperty(
    name="Darkness",
    description="Controls the darkness of the rust",
    default=0.0,
    min=-1.0,
    max=0.9,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    base_color: bpy.props.FloatVectorProperty(
    name="Base Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(1.0, 1.0, 1.0, 1.0)
    
)
    grid_main_color: bpy.props.FloatVectorProperty(
    name="Main Floor Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(0.1, 0.1, 0.1, 1.0),
    description="Base color for the floor area"
)

    grid_line_color: bpy.props.FloatVectorProperty(
    name="Grid Line Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(1.0, 1.0, 1.0, 1.0),
    description="Color of the grid lines"
)
    grid_floor_scale: bpy.props.FloatProperty(
    name="Grid Scale",
    description="Adjusts the scale of the grid pattern",
    default=4.0,
    min=2.0,
    max=50.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    backdrop_color: bpy.props.FloatVectorProperty(
    name="Backdrop Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(1.0, 1.0, 1.0, 1.0)
    
)

    
    backdrop_metallic: bpy.props.FloatProperty(
    name="Metallic",
    default=0.0,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
    )
    
    backdrop_roughness: bpy.props.FloatProperty(
    name="Roughness",
    default=1.0,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
    )

    metal_crispness: bpy.props.FloatProperty(
    name="Surface Roughness",
    description="Controls how smooth or rough the metal looks (slider 0 of ColorRamp)",
    default=0.25,  # Mid value between 0 and 0.5
    min=0.0,
    max=0.5,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    
    material_type: bpy.props.EnumProperty(
        name="Material Type",
        items=[
            ("NONE", "None", ""),
            ("METAL", "Metal", ""),
            ("SPECIAL_MAT", "Special Mat", ""),
            ("PLASTIC", "Plastic", ""),
            ("WOOD", "Wood", ""),
            ("GLASS", "Glass", ""),
            ("EMISSION", "Emission", ""),
            ("RUBBER", "Rubber", ""),
            ("FLOOR", "Floor", ""),
            ],
            default="NONE"
    )
    def get_imperfection_items(self, context):
        props = context.scene.matgen_props
        mat_type = props.material_type
        items = [("NONE", "None", "No imperfection")]

        # PLASTIC gets wear, dirt, oxidation
        if mat_type == "PLASTIC":
            items += [
                ("EDGE_WEAR", "Edge Wear", "Wear along corners"),
                ("DIRT", "Dirt", "Add grunge/dirt overlays"),
                ("OXIDATION", "Oxidation", "Oxide shading"),
                ("DOWN_DIRT", "Down Dirt", "Add vertical dirt dripping effect"),
                ("SCRATCH", "Scratch", ""),
                ]

        # METAL (NONE) gets everything
        elif mat_type == "METAL" and props.metal_type == "NONE":
            items += [
                ("EDGE_WEAR", "Edge Wear", "Wear along corners"),
                ("DIRT", "Dirt", "Add grunge/dirt overlays"),
                ("OXIDATION", "Oxidation", "Oxide shading"),
                ("RUST", "Rust", "Metal rust effect"),
                ("DOWN_DIRT", "Down Dirt", "Add vertical dirt dripping effect"),
                ("SCRATCH", "Scratch", ""),
                ]

        # METAL (typed) gets only RUST
        elif mat_type == "METAL":
            items.append(("RUST", "Rust", "Metal rust effect"))

        # RUBBER (NONE) gets only OXIDATION
        elif mat_type == "RUBBER" and props.rubber_type == "NONE":
            items.append(("OXIDATION", "Oxidation", "Oxide shading"))
        
        elif mat_type == "SPECIAL_MAT" and props.special_metal_type == "CAMO":
            items.append(("DIRT", "Dirt", "Add dirt imperfection to camo material"))


        return items



    imperfections: bpy.props.EnumProperty(
    name="Imperfection",
    description="Apply surface damage or wear",
    items=get_imperfection_items
)


    # Edge Wear Color
    edge_wear_color: bpy.props.FloatVectorProperty(
    name="Edge Wear Color",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(0.5, 0.5, 0.5, 1.0),
    
)

    
    metal_type: bpy.props.EnumProperty(
        name="Metal Type",
        items=[
            ("NONE", "None", ""),
            ("STEEL", "Steel", ""),
            ("DARK_STEEL", "Dark Steel", ""),
            ("DARK_SHINY", "Dark Shiny", ""),
            ("BRUSHED", "Brushed Metal", ""),
           
            
        ],
        default="NONE"
    )
    plastic_roughness: bpy.props.FloatProperty(
    name="Plastic Roughness",
    description="Controls the roughness of the plastic material (higher = matte, lower = shiny)",
    default=0.5,
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()  # ✅ Auto-update
)
    plastic_glossiness: bpy.props.FloatProperty(
    name="Plastic Glossiness",
    description="Controls the glossiness of the plastic material using BSDF Coat",
    default=0.0,  # Default: No gloss by default
    min=0.0,
    max=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()  # ✅ Auto-update in real time
    )
    
    emission_strength: bpy.props.FloatProperty(
        name="Emission Strength",
        description="Adjusts the intensity of the emission effect",
        default=5.0,  # 🔥 Default emission intensity
        min=0.0,
        max=5.0,  # 💡 Max intensity for strong glow
        update=lambda self, ctx: bpy.ops.material.generate_smart()  # ✅ Auto-update in real time
    )
    
    glass_base_color: bpy.props.FloatVectorProperty(
    name="Glass Color",
    description="Base color of the glass",
    subtype='COLOR',
    size=4,
    min=0.0,
    max=1.0,
    default=(1.0, 1.0, 1.0, 1.0)
)
    glass_roughness: bpy.props.FloatProperty(
    name="Glass Roughness",
    description="Adjust the roughness level of the glass surface",
    min=0.0,
    max=1.0,
    default=0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    smudge_strength: bpy.props.FloatProperty(
    name="Smudge Imperfection",
    description="Controls how much the smudge affects the glass",
    min=0.0,
    max=1.0,
    default=0.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    glass_crack_strength: bpy.props.FloatProperty(
    name="Crack Amount",
    description="Controls the intensity of cracks on the glass surface",
    min=0.0,
    max=0.6,
    default=0.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

    glass_crack_scale: bpy.props.FloatProperty(
    name="Crack Scale",
    description="Scale of the crack imperfection mapping",
    min=0.1,
    max=5.0,
    default=1.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)
    glass_texture_scale: bpy.props.FloatProperty(
    name="Texture Scale",
    description="Controls overall scale of glass texture coordinates",
    min=0.1,
    max=5.0,
    default=0.0,
    update=lambda self, ctx: bpy.ops.material.generate_smart()
)

             
    floor_type: bpy.props.EnumProperty(
    name="Floor Type",
    items=[
        ("GRID", "Grid Floor", ""),
        ("STATIC_BACKDROP", "Static Backdrop", ""),
        ("CONCRETE", "Concrete", ""), 

    ],
    default="GRID"
)

    wood_type: bpy.props.EnumProperty(
        name="Wood Type",
        items=[
            ("DEFAULT", "Default", ""),
            ("DARK", "Dark Wood", ""),
            ("LIGHT", "Light Wood", ""),
        ],
        default="DEFAULT"
        )
        
    special_mat_type: bpy.props.EnumProperty(
    name="Special Mat Type",
    items=[
            ("GOLD", "Gold", ""),
            ("COPPER", "Copper", ""),
            ("ALUMINUM", "Aluminum", ""),
            ("SILVER", "Silver", ""),
            ("CAR_PAINT", "Car Paint", "Automotive multilayer metallic material"),
            ("CAMO", "Camo", ""),
            ("SHADY", "Shady", "Stylized soft-shaded material"),
    ],
    default="GOLD"
)
    
    rubber_type: bpy.props.EnumProperty(
    name="Rubber Type",
    items=[
        ("NONE", "None", ""),
        ("NORMAL", "Normal", ""),
    ],
    default="NONE"
)

class OT_FixRustTransform(bpy.types.Operator):
    bl_idname = "material.fix_rust_transform"
    bl_label = "Fix Rust Scale & Rotation"
    bl_description = "Apply transforms and refresh rust imperfection"
    bl_options = {'REGISTER', 'UNDO'}

    def execute(self, context):
        obj = context.active_object

        if obj is None or obj.type != 'MESH':
            self.report({'WARNING'}, "No mesh object selected.")
            return {'CANCELLED'}

        # 🔧 Apply scale and rotation like Ctrl + A
        bpy.ops.object.transform_apply(location=False, rotation=True, scale=True)

        # 🔁 Nudge to trigger material refresh
        props = context.scene.matgen_props
        props.rust_spread += 0.0001

        self.report({'INFO'}, "Rust transform applied and refreshed.")
        return {'FINISHED'}


class OT_FixEdgeWearTransform(bpy.types.Operator):
    bl_idname = "material.fix_edgewear_transform"
    bl_label = "Fix Edge Wear Scale & Rotation"
    bl_description = "Applies scale/rotation and refreshes edge wear"
    bl_options = {'REGISTER', 'UNDO'}

    def execute(self, context):
        obj = context.active_object

        if obj is None or obj.type != 'MESH':
            self.report({'WARNING'}, "No mesh object selected.")
            return {'CANCELLED'}

        # 🔧 Apply scale and rotation (like Ctrl + A)
        bpy.ops.object.transform_apply(location=False, rotation=True, scale=True)

        # 🔁 Slightly adjust to trigger material update
        props = context.scene.matgen_props
        props.edge_wear_spread += 0.0001

        self.report({'INFO'}, "Edge Wear reset: scale and rotation applied.")
        return {'FINISHED'}



class OT_RemoveMaterial(bpy.types.Operator):
    bl_idname = "material.reset_material"
    bl_label = "Remove Material"
    bl_description = "Removes the material from selected objects and deletes it"

    def execute(self, context):
        selected_objs = [obj for obj in context.selected_objects if obj.type == 'MESH']
        if not selected_objs:
            self.report({'WARNING'}, "No mesh objects selected.")
            return {'CANCELLED'}

        for obj in selected_objs:
            mat = obj.active_material
            if mat:
                mat_name = mat.name
                obj.data.materials.clear()

                if mat.users == 0 and mat_name in bpy.data.materials:
                    bpy.data.materials.remove(bpy.data.materials[mat_name], do_unlink=True)

        self.report({'INFO'}, "Material(s) removed from selected object(s).")
        return {'FINISHED'}


class OT_GenerateMaterial(bpy.types.Operator):
    bl_idname = "material.generate_smart"
    bl_label = "Generate Material"

    def execute(self, context):
        obj = context.active_object
        if obj is None or obj.type != 'MESH':
            self.report({'WARNING'}, "No object selected. Cannot assign material.")
            return {'CANCELLED'}

        props = context.scene.matgen_props
        base_color = props.base_color
        edge_wear_color = props.edge_wear_color
        dirt_color = props.dirt_color

        if props.material_type == "NONE":
            obj.data.materials.clear()
            self.report({'WARNING'}, "Select any material, kidoo!")
            return {'CANCELLED'}

        r, g, b, a = [round(c * 255) for c in base_color]
        rgb_hex = f"{r:02X}{g:02X}{b:02X}"

        er, eg, eb, ea = [round(c * 255) for c in edge_wear_color]
        edge_hex = f"{er:02X}{eg:02X}{eb:02X}"

        dr, dg, db, da = [round(c * 255) for c in dirt_color]
        dirt_hex = f"{dr:02X}{dg:02X}{db:02X}"

        mat_name = f"{props.material_type}_{rgb_hex}_{edge_hex}_{dirt_hex}_{props.metal_type if props.material_type != 'SPECIAL_METAL' else props.special_metal_type}_{props.wood_type}_{props.rubber_type}_{props.imperfections}"

        if props.use_unique_material:
            mat_name += f"_{obj.name}"

        existing_mat = bpy.data.materials.get(mat_name)
        current_mat = obj.active_material
        if current_mat and current_mat.name != mat_name and current_mat.users == 1:
            mat = current_mat.copy()
            mat.name = mat_name
        elif existing_mat:
            mat = existing_mat
        else:
            mat = bpy.data.materials.new(name=mat_name)
            mat.use_nodes = True

        nodes = mat.node_tree.nodes
        links = mat.node_tree.links

        if mat.users <= 1:
            nodes.clear()

        output = nodes.get("Material Output")
        if output is None:
            output = nodes.new(type="ShaderNodeOutputMaterial")
            output.location = (400, 0)
            
        def apply_imperfection(imperf_type, mat, nodes, links, position=(0, 0)):
            # Basic placeholder for now
            if imperf_type == "DIRT":
                # Add Dirt nodes
                dirt = nodes.new("ShaderNodeTexNoise")
                dirt.location = position
                dirt.inputs["Scale"].default_value = 4.0
                color = nodes.new("ShaderNodeRGB")
                color.location = (position[0] - 200, position[1])
                color.outputs[0].default_value = props.dirt_color
                mix = nodes.new("ShaderNodeMixRGB")
                mix.location = (position[0] + 200, position[1])
                mix.inputs["Fac"].default_value = 0.5
                mix.blend_type = 'MIX'

                links.new(dirt.outputs["Fac"], mix.inputs["Fac"])
                links.new(color.outputs["Color"], mix.inputs["Color1"])

                # Plug into Base Color (you can modify as per your structure)
                base_bsdf = next((n for n in nodes if n.type == "BSDF_PRINCIPLED"), None)
                if base_bsdf:
                    links.new(mix.outputs["Color"], base_bsdf.inputs["Base Color"])

            elif imperf_type == "RUST":
                # Similar pattern: create rust nodes, position them, and link
                pass  # Fill out as needed
            # Add other imperfection types...


        if props.imperfections != "NONE":
            apply_imperfection(props.imperfections, mat, nodes, links, position=(1000, 0))

        if props.material_type == "METAL" and props.metal_type == "NONE":
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Metallic"].default_value = 1.0
            bsdf.inputs["Base Color"].default_value = base_color
            nt = nodes.new(type="ShaderNodeTexNoise")
            nt.location = (-600, -200)
            nt.inputs["Scale"].default_value = 300
            nt.inputs["Detail"].default_value = 15
            nt.inputs["Roughness"].default_value = 1.0
            nt.inputs["Distortion"].default_value = 0.3
            
            cr = nodes.new(type="ShaderNodeValToRGB")
            cr.location = (-400, -200)

            # Optional: Adjust color ramp sliders if needed
            cr.color_ramp.elements[0].position = 0.5 - (props.metal_crispness * 0.5)
            cr.color_ramp.elements[1].position = 1
            
            links.new(nt.outputs["Fac"], cr.inputs["Fac"])
            links.new(cr.outputs["Color"], bsdf.inputs["Roughness"])

            bevel = nodes.new(type="ShaderNodeBevel")
            bevel.location = (-200, -100)
            bevel.samples = 16
            bevel.inputs["Radius"].default_value = 0.022
            links.new(bevel.outputs["Normal"], bsdf.inputs["Normal"])
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
        elif props.material_type == "PLASTIC":
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Metallic"].default_value = 0.0
            bsdf.inputs["Base Color"].default_value = base_color
            bsdf.inputs["Roughness"].default_value = props.plastic_roughness
            bsdf.inputs["Coat Weight"].default_value = props.plastic_glossiness
            noise = nodes.new(type='ShaderNodeTexNoise')
            noise.location = (-600, 0)
            noise.inputs['Scale'].default_value = 40.0
            noise.inputs['Detail'].default_value = 10.0
            noise.inputs['Roughness'].default_value = 0.84
            bump = nodes.new(type='ShaderNodeBump')
            bump.location = (-300, 0)
            bump.inputs['Strength'].default_value = 0.042
            links.new(noise.outputs['Fac'], bump.inputs['Height'])
            # Connect Bump to BSDF Normal
            links.new(bump.outputs['Normal'], bsdf.inputs['Normal'])
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
        
            
       
                   
        elif props.material_type == "EMISSION":
            emis = nodes.new(type="ShaderNodeEmission")
            emis.location = (0, 0)
            emis.inputs["Color"].default_value = base_color
            emis.inputs["Strength"].default_value = props.emission_strength
            links.new(emis.outputs["Emission"], output.inputs["Surface"])
            
        elif props.material_type == "FLOOR" and props.floor_type == "GRID":
            spacing_x, spacing_y = 300, 150
            
            # Texture Coordinate → Vector Math (Scale)
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 4, 0)
            
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 3, 0)
            mapping.vector_type = 'POINT'
            links.new(tex_coord.outputs["UV"], mapping.inputs["Vector"]) 
            scale_val = props.grid_floor_scale
            mapping.inputs["Scale"].default_value = (scale_val, scale_val, scale_val)


            # Separate XYZ
            separate = nodes.new(type="ShaderNodeSeparateXYZ")
            separate.location = (-spacing_x * 2, 0)
            links.new(mapping.outputs["Vector"], separate.inputs["Vector"])

            
            # Ping Pong X
            pingpong_x = nodes.new(type="ShaderNodeMath")
            pingpong_x.location = (-spacing_x * 1, spacing_y * 1)
            pingpong_x.operation = 'PINGPONG'
            pingpong_x.inputs[1].default_value = 0.5
            links.new(separate.outputs["X"], pingpong_x.inputs[0])
            
            # Less Than X
            less_x = nodes.new(type="ShaderNodeMath")
            less_x.location = (0, spacing_y * 1)
            less_x.operation = 'LESS_THAN'
            less_x.inputs[1].default_value = 0.005
            links.new(pingpong_x.outputs[0], less_x.inputs[0])
            
            # Ping Pong Y
            pingpong_y = nodes.new(type="ShaderNodeMath")
            pingpong_y.location = (-spacing_x * 1, spacing_y * 0)
            pingpong_y.operation = 'PINGPONG'
            pingpong_y.inputs[1].default_value = 0.5
            links.new(separate.outputs["Y"], pingpong_y.inputs[0])
            
            # Less Than Y
            less_y = nodes.new(type="ShaderNodeMath")
            less_y.location = (0, spacing_y * 0)
            less_y.operation = 'LESS_THAN'
            less_y.inputs[1].default_value = 0.005
            links.new(pingpong_y.outputs[0], less_y.inputs[0])
            
            # Combine grid: Max(lt_x, lt_y)
            max1 = nodes.new(type="ShaderNodeMath")
            max1.location = (spacing_x, spacing_y * 0.5)
            max1.operation = 'MAXIMUM'
            links.new(less_x.outputs[0], max1.inputs[0])
            links.new(less_y.outputs[0], max1.inputs[1])
            
            # Soft border 1: lt3 & lt4 → min2
            lt3 = nodes.new(type="ShaderNodeMath")
            lt3.location = (0, -spacing_y * 1)
            lt3.operation = 'LESS_THAN'
            lt3.inputs[1].default_value = 0.07
            links.new(pingpong_x.outputs[0], lt3.inputs[0])
            
            lt4 = nodes.new(type="ShaderNodeMath")
            lt4.location = (0, -spacing_y * 2)
            lt4.operation = 'LESS_THAN'
            lt4.inputs[1].default_value = 0.07
            links.new(pingpong_y.outputs[0], lt4.inputs[0])
            
            min2 = nodes.new(type="ShaderNodeMath")
            min2.location = (spacing_x, -spacing_y * 1.5)
            min2.operation = 'MINIMUM'
            links.new(lt3.outputs[0], min2.inputs[0])
            links.new(lt4.outputs[0], min2.inputs[1])
            
            # Soft border 2: lt5 & lt6 → max3
            lt5 = nodes.new(type="ShaderNodeMath")
            lt5.location = (0, -spacing_y * 3)
            lt5.operation = 'LESS_THAN'
            lt5.inputs[1].default_value = 0.02
            links.new(pingpong_x.outputs[0], lt5.inputs[0])
            
            lt6 = nodes.new(type="ShaderNodeMath")
            lt6.location = (0, -spacing_y * 4)
            lt6.operation = 'LESS_THAN'
            lt6.inputs[1].default_value = 0.02
            links.new(pingpong_y.outputs[0], lt6.inputs[0])
            
            max3 = nodes.new(type="ShaderNodeMath")
            max3.location = (spacing_x, -spacing_y * 3.5)
            max3.operation = 'MAXIMUM'
            links.new(lt5.outputs[0], max3.inputs[0])
            links.new(lt6.outputs[0], max3.inputs[1])
            
            # Edge blend control: min(min2, max3) → max4
            max4 = nodes.new(type="ShaderNodeMath")
            max4.location = (spacing_x * 2, -spacing_y * 2)
            max4.operation = 'MINIMUM'
            links.new(min2.outputs[0], max4.inputs[0])
            links.new(max3.outputs[0], max4.inputs[1])
            
            # Combine mask: max(max1, max4) → max5
            max5 = nodes.new(type="ShaderNodeMath")
            max5.location = (spacing_x * 3, 0)
            max5.operation = 'MAXIMUM'
            links.new(max1.outputs[0], max5.inputs[0])
            links.new(max4.outputs[0], max5.inputs[1])
            
            # ColorRamp
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (spacing_x * 4, 0)
            color_ramp.color_ramp.elements[0].position = 0.0
            color_ramp.color_ramp.elements[1].position = 1.0
            color_ramp.color_ramp.elements[0].color = props.grid_main_color
            color_ramp.color_ramp.elements[1].color = props.grid_line_color

            links.new(max5.outputs[0], color_ramp.inputs["Fac"])
            
            # BSDF + Output
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (spacing_x * 5, 0)
            links.new(color_ramp.outputs["Color"], bsdf.inputs["Base Color"])
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
            
        elif props.material_type == "FLOOR" and props.floor_type == "STATIC_BACKDROP":
            # Texture-less simple floor material with full roughness
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Base Color"].default_value = props.backdrop_color
            bsdf.inputs["Metallic"].default_value = props.backdrop_metallic
            bsdf.inputs["Roughness"].default_value = props.backdrop_roughness
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
            
        elif props.material_type == "FLOOR" and props.floor_type == "CONCRETE":
            spacing_x, spacing_y = 300, 200

            # Create main nodes
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            output = nodes.get("Material Output") or nodes.new(type="ShaderNodeOutputMaterial")
            output.location = (spacing_x * 2, 0)
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])

            # Texture Coordinate and Mapping (main)
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 4, 0)

            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 3, 0)
            mapping.inputs["Scale"].default_value = (props.concrete_crack_scale,) * 3

            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])

            # Noise Texture 1
            noise1 = nodes.new(type="ShaderNodeTexNoise")
            noise1.location = (-spacing_x * 2, 0)
            noise1.inputs["Scale"].default_value = 7
            noise1.inputs["Detail"].default_value = 15
            noise1.inputs["Roughness"].default_value = 0.691
            links.new(mapping.outputs["Vector"], noise1.inputs["Vector"])

            # ColorRamp 1
            cr1 = nodes.new(type="ShaderNodeValToRGB")
            cr1.location = (-spacing_x, 0)
            cr1.color_ramp.elements[0].position = 0.342
            cr1.color_ramp.elements[1].position = 0.759
            cr1.color_ramp.elements[0].color = (0.135, 0.135, 0.135, 1)  # #696969FF
            cr1.color_ramp.elements[1].color = (0.497, 0.497, 0.497, 1)  # #D9D9D9FF
            
            links.new(cr1.outputs["Color"], bsdf.inputs["Base Color"])

            # Bump 1 (initial)
            bump1 = nodes.new(type="ShaderNodeBump")
            bump1.location = (spacing_x, -spacing_y)
            bump1.inputs["Strength"].default_value = 0.2
            links.new(bump1.outputs["Normal"], bsdf.inputs["Normal"])

            # Noise Texture 2
            noise2 = nodes.new(type="ShaderNodeTexNoise")
            noise2.location = (-spacing_x * 2, -spacing_y * 1.5)
            noise2.inputs["Scale"].default_value = 7
            noise2.inputs["Detail"].default_value = 10
            noise2.inputs["Roughness"].default_value = 0.5
            noise2.inputs["Distortion"].default_value = 4
            links.new(mapping.outputs["Vector"], noise2.inputs["Vector"])

            # ColorRamp CR2
            cr2 = nodes.new(type="ShaderNodeValToRGB")
            cr2.location = (-spacing_x, -spacing_y * 1.5)
            cr2.color_ramp.elements[1].position = 0.440
            links.new(noise2.outputs["Fac"], cr2.inputs["Fac"])

            # Mix Color (Difference) between noise1 and noise2
            mix = nodes.new(type="ShaderNodeMixRGB")
            mix.location = (0, -spacing_y * 1.5)
            mix.blend_type = 'DIFFERENCE'
            mix.inputs["Fac"].default_value = 0.267
            links.new(noise1.outputs["Color"], mix.inputs["Color1"])
            links.new(cr2.outputs["Color"], mix.inputs["Color2"])
            links.new(mix.outputs["Color"], bump1.inputs["Height"])
            links.new(mix.outputs["Color"], cr1.inputs["Fac"])  # ✅ Correct input to ColorRamp1

            # Voronoi Texture Setup
            tex_coord2 = nodes.new(type="ShaderNodeTexCoord")
            tex_coord2.location = (-spacing_x * 6, -spacing_y * 4)

            nt3 = nodes.new(type="ShaderNodeTexNoise")  # nt3
            nt3.location = (-spacing_x * 5, -spacing_y * 4)
            nt3.inputs["Detail"].default_value = 3.2
            nt3.inputs["Roughness"].default_value = 6
            links.new(tex_coord2.outputs["Object"], nt3.inputs["Vector"])

            mix2 = nodes.new(type="ShaderNodeMixRGB")  # mix2
            mix2.location = (-spacing_x * 4, -spacing_y * 4)
            mix2.inputs["Fac"].default_value = props.crack_detailing

            links.new(nt3.outputs["Color"], mix2.inputs["Color1"])
            links.new(tex_coord2.outputs["Object"], mix2.inputs["Color2"])

            mapping2 = nodes.new(type="ShaderNodeMapping")  # mapping2
            mapping2.location = (-spacing_x * 3, -spacing_y * 4)
            mapping2.inputs["Scale"].default_value = (props.concrete_crack_scale,) * 3

            links.new(mix2.outputs["Color"], mapping2.inputs["Vector"])

            voronoi = nodes.new(type="ShaderNodeTexVoronoi")
            voronoi.location = (-spacing_x * 2, -spacing_y * 4)
            voronoi.feature = 'DISTANCE_TO_EDGE'
            links.new(mapping2.outputs["Vector"], voronoi.inputs["Vector"])

            # ColorRamp CR3
            cr3 = nodes.new(type="ShaderNodeValToRGB")
            cr3.location = (-spacing_x, -spacing_y * 4)
            cr3.color_ramp.elements[1].position = 0.010
            links.new(voronoi.outputs["Distance"], cr3.inputs["Fac"])

            # Bump 2
            bump2 = nodes.new(type="ShaderNodeBump")
            bump2.location = (spacing_x * 0.3, -spacing_y * 1.5)
            bump2.inputs["Strength"].default_value = props.crack_strength
            links.new(cr3.outputs["Color"], bump2.inputs["Height"])
            links.new(bump2.outputs["Normal"], bump1.inputs["Normal"])

       
        elif props.material_type == "SPECIAL_MAT":
            
            if props.special_mat_type == "SILVER":
                spacing_x = 300  # Adjust horizontal spacing
                spacing_y = 150  # Adjust vertical spacing
                
                # ✅ Create BSDF Shader & Output Node
                bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)
                
                bsdf.inputs["Metallic"].default_value = 1.0  # 🔥 Ensures fully metallic Silver
                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
                # ✅ First Noise Texture & Color Ramp for Base Color
                noise_texture = nodes.new(type="ShaderNodeTexNoise")
                noise_texture.location = (-spacing_x * 2, spacing_y * 2)
                noise_texture.inputs["Scale"].default_value = 6  # Increased for finer grain
                noise_texture.inputs["Detail"].default_value = 12  # More detail for realism
                noise_texture.inputs["Roughness"].default_value = 0.85
                noise_texture.inputs["Lacunarity"].default_value = 2.5
                noise_texture.inputs["Distortion"].default_value = 1.8
                
                color_ramp = nodes.new(type="ShaderNodeValToRGB")
                color_ramp.location = (-spacing_x, spacing_y * 2)
                color_ramp.color_ramp.elements[0].position = 0.350
                color_ramp.color_ramp.elements[1].position = 0.490
                color_ramp.color_ramp.elements[0].color = (192/255, 192/255, 192/255, 1.0)  # Light Silver
                color_ramp.color_ramp.elements[1].color = (220/255, 220/255, 220/255, 1.0)  # Bright Silver
                
                links.new(noise_texture.outputs["Fac"], color_ramp.inputs["Fac"])
                links.new(color_ramp.outputs["Color"], bsdf.inputs["Base Color"])
                
                # ✅ Mapping & Texture Coordinate Setup
                tex_coord = nodes.new(type="ShaderNodeTexCoord")
                tex_coord.location = (-spacing_x * 3, spacing_y)
                
                mapping = nodes.new(type="ShaderNodeMapping")
                mapping.location = (-spacing_x * 2, spacing_y)
                
                links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
                links.new(mapping.outputs["Vector"], noise_texture.inputs["Vector"])
                
                # ✅ Second Noise Texture for Roughness (nt2)
                nt2 = nodes.new(type="ShaderNodeTexNoise")
                nt2.location = (-spacing_x * 2, 0)
                nt2.inputs["Scale"].default_value = 3.2  # Slightly finer roughness details
                nt2.inputs["Detail"].default_value = 10
                nt2.inputs["Roughness"].default_value = 1  # Lower roughness for shiny silver
                
                cr2 = nodes.new(type="ShaderNodeValToRGB")
                cr2.location = (-spacing_x, 0)
                cr2.color_ramp.elements[0].position = 0.342
                cr2.color_ramp.elements[1].position = 0.504
                cr2.color_ramp.elements[0].color = (67/255, 67/255, 66/255, 1.0)  # Mid-tone Silver
                cr2.color_ramp.elements[1].color = (127/255, 127/255, 127/255, 1.0)  # Brighter Silver
                
                links.new(nt2.outputs["Fac"], cr2.inputs["Fac"])
                links.new(cr2.outputs["Color"], bsdf.inputs["Roughness"])
                links.new(mapping.outputs["Vector"], nt2.inputs["Vector"])
                
                # ✅ Third Noise Texture for Bump (nt3)
                nt3 = nodes.new(type="ShaderNodeTexNoise")
                nt3.location = (-spacing_x * 2, -spacing_y * 1)
                nt3.inputs["Scale"].default_value = 25  # Adjusted for fine metallic grain
                nt3.inputs["Detail"].default_value = 10
                nt3.inputs["Roughness"].default_value = 0.8
                
                bump = nodes.new(type="ShaderNodeBump")
                bump.location = (-spacing_x, -spacing_y * 1)
                bump.inputs["Strength"].default_value = 0.015  # Lower bump for smoother silver
                
                links.new(nt3.outputs["Fac"], bump.inputs["Height"])
                links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
                links.new(mapping.outputs["Vector"], nt3.inputs["Vector"])

            
            elif props.special_mat_type == "ALUMINUM":
                spacing_x = 300  # Adjust horizontal spacing
                spacing_y = 150  # Adjust vertical spacing
                bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)

                bsdf.inputs["Metallic"].default_value = 1.0  # Aluminum is fully metallic
                # Base Color Nodes
                tex_coord1 = nodes.new(type="ShaderNodeTexCoord")
                tex_coord1.location = (-spacing_x * 3, spacing_y * 1)
                mapping1 = nodes.new(type="ShaderNodeMapping")
                mapping1.location = (-spacing_x * 2, spacing_y * 1)
                noise1 = nodes.new(type="ShaderNodeTexNoise")
                noise1.location = (-spacing_x, spacing_y * 2)
                noise1.inputs["Scale"].default_value = 4.0  # Higher scale for finer grainy details
                noise1.inputs["Detail"].default_value = 12.0  # More detailed texture for brushed aluminum
                noise1.inputs["Roughness"].default_value = 0.85
                noise1.inputs["Distortion"].default_value = 0.5  # Low distortion for sleek finish
                color_ramp_color = nodes.new(type="ShaderNodeValToRGB")
                color_ramp_color.location = (spacing_x, spacing_y * 3)
                color_ramp_color.color_ramp.elements[0].color = (0.75, 0.75, 0.75, 1)  # Light gray base
                color_ramp_color.color_ramp.elements[1].color = (0.9, 0.9, 0.9, 1)  # Silvery highlight
                links.new(tex_coord1.outputs["Object"], mapping1.inputs["Vector"])
                links.new(mapping1.outputs["Vector"], noise1.inputs["Vector"])
                links.new(noise1.outputs["Color"], color_ramp_color.inputs["Fac"])
                links.new(color_ramp_color.outputs["Color"], bsdf.inputs["Base Color"])
                # Metallic control (Same as other metals)
                color_ramp_metal = nodes.new(type="ShaderNodeValToRGB")
                color_ramp_metal.location = (spacing_x, spacing_y * 2)
                color_ramp_metal.color_ramp.elements[0].position = 0.2
                color_ramp_metal.color_ramp.elements[1].position = 0.65
                links.new(color_ramp_metal.outputs["Color"], bsdf.inputs["Metallic"])
                # Adjust Roughness for brushed aluminum
                tex_coord2 = nodes.new(type="ShaderNodeTexCoord")
                tex_coord2.location = (-spacing_x * 3, -spacing_y * 2)
                mapping2 = nodes.new(type="ShaderNodeMapping")
                mapping2.location = (-spacing_x * 2, -spacing_y * 2)
                noise2 = nodes.new(type="ShaderNodeTexNoise")
                noise2.location = (-spacing_x, -spacing_y * 2)
                noise2.inputs["Scale"].default_value = 6.0  # More scaling for surface detail
                noise2.inputs["Detail"].default_value = 10.0
                noise2.inputs["Roughness"].default_value = 0.7  # Higher roughness for matte finish
                color_ramp_rough = nodes.new(type="ShaderNodeValToRGB")
                color_ramp_rough.location = (spacing_x, spacing_y * 1)
                color_ramp_rough.color_ramp.elements[0].position = 0.45
                color_ramp_rough.color_ramp.elements[1].position = 0.75
                links.new(tex_coord2.outputs["Object"], mapping2.inputs["Vector"])
                links.new(mapping2.outputs["Vector"], noise2.inputs["Vector"])
                links.new(noise2.outputs["Color"], color_ramp_rough.inputs["Fac"])
                links.new(color_ramp_rough.outputs["Color"], bsdf.inputs["Roughness"])
                # Bump Mapping for fine brushed texture effect
                noise3 = nodes.new(type="ShaderNodeTexNoise")
                noise3.location = (spacing_x * 2, spacing_y * 1)
                bump = nodes.new(type="ShaderNodeBump")
                bump.location = (spacing_x * 3, spacing_y * 1)
                bump.inputs["Strength"].default_value = 0.05  # Subtle texture for fine lines in brushed aluminum
                links.new(noise3.outputs["Color"], bump.inputs["Height"])
                links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
                links.new(bsdf.outputs['BSDF'], output.inputs['Surface'])
            
            elif props.special_mat_type == "COPPER":
                # === Base Nodes ===
                bsdf = nodes.new("ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)
                bsdf.inputs["Metallic"].default_value = 1.0
                bsdf.inputs["Roughness"].default_value = 0.1
                bsdf.inputs["Base Color"].default_value = (0.679, 0.270, 0.196, 1.0)  # yellow-orange

                
                output = nodes.get("Material Output") or nodes.new("ShaderNodeOutputMaterial")
                output.location = (400, 0)
                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
                
                # === Texture Coordinate & Mapping ===
                tex_coord = nodes.new("ShaderNodeTexCoord")
                tex_coord.location = (-1600, 0)
                
                mapping = nodes.new("ShaderNodeMapping")
                mapping.location = (-1400, 0)
                links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
                
                # === Noise Texture → Map Range → Roughness (Layer 1) ===
                noise1 = nodes.new("ShaderNodeTexNoise")
                noise1.location = (-1200, 0)
                noise1.inputs["Scale"].default_value = 600
                links.new(mapping.outputs["Vector"], noise1.inputs["Vector"])
                
                map_range = nodes.new("ShaderNodeMapRange")
                map_range.location = (-1000, 0)
                map_range.inputs["To Min"].default_value = 0.28
                map_range.inputs["To Max"].default_value = 0.380
                links.new(noise1.outputs["Fac"], map_range.inputs["Value"])
                
                # === Voronoi → Separate RGB → Greater Than ===
                voronoi = nodes.new("ShaderNodeTexVoronoi")
                voronoi.location = (-1200, -300)
                voronoi.feature = 'SMOOTH_F1'
                voronoi.inputs["Scale"].default_value = 15
                links.new(mapping.outputs["Vector"], voronoi.inputs["Vector"])
                
                separate_rgb = nodes.new("ShaderNodeSeparateRGB")
                separate_rgb.location = (-1000, -300)
                links.new(voronoi.outputs["Color"], separate_rgb.inputs["Image"])
                
                greater_than = nodes.new("ShaderNodeMath")
                greater_than.location = (-800, -300)
                greater_than.operation = 'GREATER_THAN'
                greater_than.inputs[1].default_value = 0.800
                links.new(separate_rgb.outputs["R"], greater_than.inputs[0])
                
                # === Mix1 (Subtract) ===
                mix1 = nodes.new("ShaderNodeMixRGB")
                mix1.location = (-600, -100)
                mix1.blend_type = 'SUBTRACT'
                mix1.inputs["Fac"].default_value = 0.058
                links.new(map_range.outputs["Result"], mix1.inputs["Color1"])
                links.new(greater_than.outputs["Value"], mix1.inputs["Color2"])
                
                # === Noise2 → Mix2 (Linear Light) ===
                noise2 = nodes.new("ShaderNodeTexNoise")
                noise2.location = (-1200, -600)
                links.new(mapping.outputs["Vector"], noise2.inputs["Vector"])
                
                mix2 = nodes.new("ShaderNodeMixRGB")
                mix2.location = (-900, -600)
                mix2.blend_type = 'LINEAR_LIGHT'
                mix2.inputs["Fac"].default_value = 0.092
                links.new(tex_coord.outputs["Object"], mix2.inputs["Color1"])
                links.new(noise2.outputs["Color"], mix2.inputs["Color2"])
                
                # === Noise3 → Mix3 (Overlay) ===
                noise3 = nodes.new("ShaderNodeTexNoise")
                noise3.location = (-1200, -900)
                links.new(voronoi.outputs["Position"], noise3.inputs["Vector"])
                
                mix3 = nodes.new("ShaderNodeMixRGB")
                mix3.location = (-600, -600)
                mix3.blend_type = 'OVERLAY'
                mix3.inputs["Fac"].default_value = 0.633
                links.new(mix1.outputs["Color"], mix3.inputs["Color1"])
                links.new(noise3.outputs["Fac"], mix3.inputs["Color2"])
                
                # Connect to BSDF Roughness
                links.new(mix3.outputs["Color"], bsdf.inputs["Roughness"])
                
                # === Noise4 → Mix4 (Overlay) ===
                noise4 = nodes.new("ShaderNodeTexNoise")
                noise4.location = (-1200, -1200)
                noise4.inputs["Detail"].default_value = 15
                noise4.inputs["Roughness"].default_value = 1.0
                links.new(mapping.outputs["Vector"], noise4.inputs["Vector"])
                
                mix4 = nodes.new("ShaderNodeMixRGB")
                mix4.location = (-300, -600)
                mix4.blend_type = 'OVERLAY'
                mix4.inputs["Fac"].default_value = 0.633
                links.new(mix3.outputs["Color"], mix4.inputs["Color1"])
                links.new(noise4.outputs["Fac"], mix4.inputs["Color2"])
                
                # Final Roughness connection
                links.new(mix4.outputs["Color"], bsdf.inputs["Roughness"])
                
                # === Bump Mapping with Noise5 ===
                noise5 = nodes.new("ShaderNodeTexNoise")
                noise5.location = (-1000, -1500)
                noise5.inputs["Scale"].default_value = 20
                noise5.inputs["Detail"].default_value = 8
                noise5.inputs["Roughness"].default_value = 0.8
                links.new(mapping.outputs["Vector"], noise5.inputs["Vector"])
                bump = nodes.new("ShaderNodeBump")
                bump.location = (-600, -1500)
                bump.inputs["Strength"].default_value = 0.09
                links.new(noise5.outputs["Fac"], bump.inputs["Height"])
                links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])

                
            elif props.special_mat_type == "CAR_PAINT":
                # Car Paint Material Setup

                # Principled BSDF + Output
                bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
                bsdf.inputs["Base Color"].default_value = (*props.car_paint_color, 1.0)
                bsdf.location = (0, 0)
                bsdf.inputs["Metallic"].default_value = 1.0
                bsdf.inputs["Coat Weight"].default_value = 1.0  # ✅ Works in Blender 4.4+


                output = nodes.get("Material Output") or nodes.new(type="ShaderNodeOutputMaterial")
                output.location = (400, 0)
                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])

                # Texture Coordinate
                tex_coord = nodes.new(type="ShaderNodeTexCoord")
                tex_coord.location = (-1000, 0)

                # Mapping
                mapping = nodes.new(type="ShaderNodeMapping")
                mapping.location = (-800, 0)
                links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])

                # Voronoi Texture
                voronoi = nodes.new(type="ShaderNodeTexVoronoi")
                voronoi.location = (-600, 0)
                voronoi.feature = 'F1'
                voronoi.distance = 'EUCLIDEAN'
                voronoi.inputs["Scale"].default_value = 650
                links.new(mapping.outputs["Vector"], voronoi.inputs["Vector"])

                # ColorRamp between Voronoi and BSDF Roughness
                cr = nodes.new(type="ShaderNodeValToRGB")
                cr.location = (-400, 0)
                cr.color_ramp.elements[0].color = (0.4, 0.4, 0.4, 1.0)    # grey
                cr.color_ramp.elements[1].color = (0.8, 0.8, 0.8, 1.0)    # bright grey
                links.new(voronoi.outputs["Color"], cr.inputs["Fac"])
                links.new(cr.outputs["Color"], bsdf.inputs["Roughness"])
                
            elif props.special_mat_type == "SHADY":
                # === BSDF + Output ===
                bsdf = nodes.new("ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)
                bsdf.inputs["Metallic"].default_value = 0.0
                bsdf.inputs["Roughness"].default_value = 0.5  # Optional tweak
                output = nodes.get("Material Output") or nodes.new("ShaderNodeOutputMaterial")
                output.location = (400, 0)
                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
                
                # === Brick Texture ===
                brick = nodes.new("ShaderNodeTexBrick")
                brick.location = (-600, 0)
                
                # === Noise Texture → Brick Vector ===
                noise = nodes.new("ShaderNodeTexNoise")
                noise.location = (-900, 0)
                noise.inputs["Scale"].default_value = 5.0
                noise.inputs["Detail"].default_value = 2.0
                
                # === Mapping & Texture Coordinate → Noise
                tex_coord = nodes.new("ShaderNodeTexCoord")
                tex_coord.location = (-1300, 0)

                mapping = nodes.new("ShaderNodeMapping")
                mapping.location = (-1100, 0)
                mapping.inputs["Scale"].default_value = (props.shady_scale,) * 3

                links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
                links.new(mapping.outputs["Vector"], noise.inputs["Vector"])

                # === Connect noise → brick vector
                links.new(noise.outputs["Fac"], brick.inputs["Vector"])
                
                # === ColorRamp (Constant) ===
                ramp = nodes.new("ShaderNodeValToRGB")
                ramp.location = (-300, 0)
                ramp.color_ramp.interpolation = 'CONSTANT'
                
                # Add third slider
                while len(ramp.color_ramp.elements) < 3:
                    ramp.color_ramp.elements.new(0.672)
                    
                # Set colors and positions
                ramp.color_ramp.elements[1].color = props.shady_color_1
                ramp.color_ramp.elements[0].color = props.shady_color_2
                ramp.color_ramp.elements[2].color = props.shady_color_3

                ramp.color_ramp.elements[1].position = 0.490
                ramp.color_ramp.elements[2].position = 0.672
                
                # Connect Brick → Ramp → BSDF Base Color
                links.new(brick.outputs["Color"], ramp.inputs["Fac"])
                links.new(ramp.outputs["Color"], bsdf.inputs["Base Color"])
                
                # === Bump Mapping ===
                bump = nodes.new("ShaderNodeBump")
                bump.location = (-300, -200)
                bump.inputs["Strength"].default_value = 0.6
                links.new(brick.outputs["Color"], bump.inputs["Height"])
                links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])

                
                
            elif props.special_mat_type == "CAMO":
                spacing_x = 300
                spacing_y = 150

                # Output & BSDF
                bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)
                output = nodes.get("Material Output") or nodes.new(type="ShaderNodeOutputMaterial")
                output.location = (spacing_x * 2, 0)
                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])

                # ColorRamp → BSDF Base Color
                cr = nodes.new(type="ShaderNodeValToRGB")
                cr.location = (-spacing_x, 0)
                cr.color_ramp.interpolation = 'CONSTANT'
                cr.color_ramp.elements[0].position = 0.467
                cr.color_ramp.elements[0].color = (0.384, 0.502, 0.231, 1)  # #62803B
                cr.color_ramp.elements[1].position = 0.6
                cr.color_ramp.elements[1].color = (0.78, 1.0, 0.47, 1)     # #C7FF78
                cr.color_ramp.elements.new(0.75)
                cr.color_ramp.elements[2].color = (0.243, 0.302, 0.161, 1)  # #3E4D29
                cr.color_ramp.elements.new(0.9)
                cr.color_ramp.elements[3].color = (0.624, 0.8, 0.376, 1)    # #9FCC60
                links.new(cr.outputs["Color"], bsdf.inputs["Base Color"])
                
                # Set Camo Colors based on camo_type
                if props.camo_type == "TYPE1":
                    colors = ["#62803B", "#C7FF78", "#3E4D29", "#9FCC60"]
                elif props.camo_type == "TYPE2":
                    colors = [ "#64735A", "#A4A67C", "#3D402C", "#8B8C65",  ]  # desert
                elif props.camo_type == "TYPE3":
                    colors = ["#4A4A4A", "#8C8C8C", "#2F2F2F", "#5C5C5C"]  # urban grey
                elif props.camo_type == "TYPE4":
                    colors = ["#58592D", "#8C7735", "#59422E", "#262321"]  # Saratoga
                else:
                    colors = ["#62803B", "#C7FF78", "#3E4D29", "#9FCC60"]  # fallback

                # Convert hex to RGBA
                def hex_to_rgba(hex_code):
                    return tuple(int(hex_code[i:i+2], 16) / 255.0 for i in (1, 3, 5)) + (1.0,)

                for i, hex_color in enumerate(colors):
                    cr.color_ramp.elements[i].color = hex_to_rgba(hex_color)


                # Voronoi Texture
                voronoi = nodes.new(type="ShaderNodeTexVoronoi")
                voronoi.location = (-spacing_x * 2, 0)
                voronoi.voronoi_dimensions = '4D'
                voronoi.feature = 'SMOOTH_F1'
                voronoi.distance = 'MINKOWSKI'
                voronoi.inputs["W"].default_value = props.pattern_randomness
                voronoi.inputs["Scale"].default_value = 5.0
                voronoi.inputs["Exponent"].default_value = 1.5
                links.new(voronoi.outputs["Distance"], cr.inputs["Fac"])

                # Mapping + TexCoord → Voronoi
                mapping = nodes.new(type="ShaderNodeMapping")
                mapping.location = (-spacing_x * 3, 0)
                mapping.inputs["Scale"].default_value = (
                    props.camo_scale,
                    props.camo_scale,
                    props.camo_scale
                )

                texcoord = nodes.new(type="ShaderNodeTexCoord")
                texcoord.location = (-spacing_x * 4, spacing_y * 0.5)

                # Noise → MixRGB → Mapping Vector
                noise = nodes.new(type="ShaderNodeTexNoise")
                noise.location = (-spacing_x * 4, -spacing_y * 0.5)
                noise.inputs["Scale"].default_value = 5.0
                noise.inputs["Detail"].default_value = 4.0
                noise.inputs["Roughness"].default_value = 0.5
                noise.inputs["Distortion"].default_value = 0.6

                mix = nodes.new(type="ShaderNodeMixRGB")
                mix.location = (-spacing_x * 3.5, 0)
                mix.blend_type = 'MIX'
                mix.inputs["Fac"].default_value = 0.7
                links.new(noise.outputs["Color"], mix.inputs["Color1"])
                links.new(texcoord.outputs["Object"], mix.inputs["Color2"])

                links.new(mix.outputs["Color"], mapping.inputs["Vector"])
                links.new(mapping.outputs["Vector"], voronoi.inputs["Vector"])

                # CR2 → BSDF Roughness
                cr2 = nodes.new(type="ShaderNodeValToRGB")
                cr2.location = (-spacing_x * 1.5, -spacing_y * 1.5)
                cr2.color_ramp.interpolation = 'LINEAR'
                cr2.color_ramp.elements[0].position = 0.43
                cr2.color_ramp.elements[1].position = 0.53
                cr2.color_ramp.elements[0].color = (0.5, 0.5, 0.5, 1)  # grey
                links.new(cr2.outputs["Color"], bsdf.inputs["Roughness"])

                # Noise Texture nt2 → CR2 Fac
                nt2 = nodes.new(type="ShaderNodeTexNoise")
                nt2.location = (-spacing_x * 2.5, -spacing_y * 1.5)
                nt2.inputs["Scale"].default_value = 3.0

                mapping2 = nodes.new(type="ShaderNodeMapping")
                mapping2.location = (-spacing_x * 3.2, -spacing_y * 1.5)
                texcoord2 = nodes.new(type="ShaderNodeTexCoord")
                texcoord2.location = (-spacing_x * 4, -spacing_y * 1.5)

                links.new(texcoord2.outputs["Object"], mapping2.inputs["Vector"])
                links.new(mapping2.outputs["Vector"], nt2.inputs["Vector"])
                links.new(nt2.outputs["Fac"], cr2.inputs["Fac"])

                # Bump node
                bump = nodes.new(type="ShaderNodeBump")
                bump.location = (-spacing_x * 0.5, -spacing_y)
                bump.inputs["Strength"].default_value = 0.01
                links.new(cr2.outputs["Color"], bump.inputs["Height"])
                links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
                
             # Gold Base Color Test: Simple approach to check material assignment
            elif props.gold_type == "TYPE1":
                spacing_x = 300  # Adjust horizontal spacing
                spacing_y = 150  # Adjust vertical spacing
                
                # ✅ Create BSDF Shader & Output Node
                bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)
                
                bsdf.inputs["Metallic"].default_value = 1.0  # 🔥 Ensures fully metallic Gold
                
                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
                # ✅ First Noise Texture & Color Ramp for Base Color
                noise_texture = nodes.new(type="ShaderNodeTexNoise")
                noise_texture.location = (-spacing_x * 2, spacing_y * 2)
                noise_texture.inputs["Scale"].default_value = 4
                noise_texture.inputs["Detail"].default_value = 8
                noise_texture.inputs["Roughness"].default_value = 0.95
                noise_texture.inputs["Lacunarity"].default_value = 2.8
                noise_texture.inputs["Distortion"].default_value = 2.3
                
                color_ramp = nodes.new(type="ShaderNodeValToRGB")
                color_ramp.location = (-spacing_x, spacing_y * 2)
                color_ramp.color_ramp.elements[0].position = 0.250
                color_ramp.color_ramp.elements[1].position = 0.490
                color_ramp.color_ramp.elements[0].color = (168/255, 84/255, 0/255, 1.0) # A67E00FF
                color_ramp.color_ramp.elements[1].color = (255/255, 224/255, 4/255, 1)   # FFE004FF
                
                links.new(noise_texture.outputs["Fac"], color_ramp.inputs["Fac"])
                links.new(color_ramp.outputs["Color"], bsdf.inputs["Base Color"])
                
                # ✅ Mapping & Texture Coordinate Setup
                tex_coord = nodes.new(type="ShaderNodeTexCoord")
                tex_coord.location = (-spacing_x * 3, spacing_y)
                
                mapping = nodes.new(type="ShaderNodeMapping")
                mapping.location = (-spacing_x * 2, spacing_y)
                
                links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
                links.new(mapping.outputs["Vector"], noise_texture.inputs["Vector"])
                
                # ✅ Second Noise Texture for Roughness (nt2)
                nt2 = nodes.new(type="ShaderNodeTexNoise")
                nt2.location = (-spacing_x * 2, 0)
                nt2.inputs["Scale"].default_value = 4
                nt2.inputs["Detail"].default_value = 5
                nt2.inputs["Roughness"].default_value = 0.43
                
                cr2 = nodes.new(type="ShaderNodeValToRGB")
                cr2.location = (-spacing_x, 0)

                # Ensure elements exist
                while len(cr2.color_ramp.elements) < 2:
                    cr2.color_ramp.elements.new(1.0)

                elem0 = cr2.color_ramp.elements[0]
                elem1 = cr2.color_ramp.elements[1]

                # Set fixed values
                elem0.position = 0.242
                elem0.color = (15/255, 15/255, 15/255, 1.0)  # 474747FF

                # Set mapped position
                mapped_value = 0.48
                mapped_value = max(0.33, min(1.0, mapped_value))
                elem1.position = mapped_value
                elem1.color = (69/255, 69/255, 69/255, 1.0)  # 858585FF

                
                links.new(nt2.outputs["Fac"], cr2.inputs["Fac"])
                links.new(cr2.outputs["Color"], bsdf.inputs["Roughness"])
                links.new(mapping.outputs["Vector"], nt2.inputs["Vector"])
                
                # ✅ Third Noise Texture for Bump (nt3)
                nt3 = nodes.new(type="ShaderNodeTexNoise")
                nt3.location = (-spacing_x * 2, -spacing_y * 1)
                nt3.inputs["Scale"].default_value = 30
                nt3.inputs["Detail"].default_value = 10
                nt3.inputs["Roughness"].default_value = 0.8
                
                bump = nodes.new(type="ShaderNodeBump")
                bump.location = (-spacing_x, -spacing_y * 1)
                bump.inputs["Strength"].default_value = 0.025
                
                links.new(nt3.outputs["Fac"], bump.inputs["Height"])
                links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
                links.new(mapping.outputs["Vector"], nt3.inputs["Vector"])
                
            elif props.gold_type == "TYPE2":
                spacing_x = 300
                spacing_y = 150

                # Output & BSDF
                bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
                bsdf.location = (0, 0)
                bsdf.inputs["Metallic"].default_value = 1.0
                bsdf.inputs["Base Color"].default_value = (1.0, 0.724, 0.054, 1.0)  # yellow-orange

                links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])

                # TexCoord and Mapping
                tex_coord = nodes.new(type="ShaderNodeTexCoord")
                tex_coord.location = (-spacing_x * 3, 0)
                mapping = nodes.new(type="ShaderNodeMapping")
                mapping.location = (-spacing_x * 2, 0)
                links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])

                # Noise 1
                nt1 = nodes.new(type="ShaderNodeTexNoise")
                nt1.location = (-spacing_x * 2, spacing_y)
                nt1.inputs["Scale"].default_value = 4.5
                nt1.inputs["Detail"].default_value = 15.0
                nt1.inputs["Roughness"].default_value = 0.8
                nt1.inputs["Distortion"].default_value = 0.2
                links.new(mapping.outputs["Vector"], nt1.inputs["Vector"])

                # ColorRamp 1 → Bump1
                cr1 = nodes.new(type="ShaderNodeValToRGB")
                cr1.location = (-spacing_x, spacing_y)
                cr1.color_ramp.elements[1].position = 0.480
                links.new(nt1.outputs["Fac"], cr1.inputs["Fac"])

                bump1 = nodes.new(type="ShaderNodeBump")
                bump1.location = (0, spacing_y)
                bump1.inputs["Strength"].default_value = 0.150
                links.new(cr1.outputs["Color"], bump1.inputs["Height"])

                # Noise 2 → Bump2
                nt2 = nodes.new(type="ShaderNodeTexNoise")
                nt2.location = (-spacing_x * 2, -spacing_y)
                nt2.inputs["Scale"].default_value = 30
                nt2.inputs["Detail"].default_value = 10
                nt2.inputs["Roughness"].default_value = 0.8
                nt2.inputs["Distortion"].default_value = 2.0
                links.new(mapping.outputs["Vector"], nt2.inputs["Vector"])

                bump2 = nodes.new(type="ShaderNodeBump")
                bump2.location = (spacing_x, 0)
                bump2.inputs["Strength"].default_value = 0.02
                links.new(bump1.outputs["Normal"], bump2.inputs["Normal"])
                links.new(nt2.outputs["Fac"], bump2.inputs["Height"])
                links.new(bump2.outputs["Normal"], bsdf.inputs["Normal"])

                # ColorRamp 2 → Roughness
                cr2 = nodes.new(type="ShaderNodeValToRGB")
                cr2.location = (-spacing_x, -spacing_y)
                cr2.color_ramp.elements[0].position = 0.174
                cr2.color_ramp.elements[0].color = (0.584, 0.584, 0.584, 1)  # #959595FF
                cr2.color_ramp.elements[1].position = 0.677
                cr2.color_ramp.elements[1].color = (0.328, 0.328, 0.328, 1)

                links.new(nt1.outputs["Fac"], cr2.inputs["Fac"])
                links.new(cr2.outputs["Color"], bsdf.inputs["Roughness"])
                
        elif props.material_type == "METAL" and props.metal_type == "BRUSHED":
            # Brushed Metal Shader Setup
            # === Base Setup ===
            bsdf = nodes.new("ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Metallic"].default_value = 1.0
            bsdf.inputs["Roughness"].default_value = 0.361
            output = nodes.get("Material Output")
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
            
            # === Texture Coordinate & Mapping ===
            texcoord = nodes.new("ShaderNodeTexCoord")
            texcoord.location = (-1000, 0)
            
            mapping = nodes.new("ShaderNodeMapping")
            mapping.location = (-800, 0)
            links.new(texcoord.outputs["Object"], mapping.inputs["Vector"])
            # === Brushed Type Selection ===
            if props.brushed_type == "TYPE1":
                mapping.inputs["Scale"].default_value = (4.0, 4.0, 180)
                
            # === Noise Texture (Brushed Grain) ===
            noise = nodes.new("ShaderNodeTexNoise")
            noise.location = (-600, 0)
            noise.inputs["Detail"].default_value = 15
            noise.inputs["Roughness"].default_value = 0.6
            noise.inputs["Distortion"].default_value = 4.0
            links.new(mapping.outputs["Vector"], noise.inputs["Vector"])
            
            # === ColorRamp (cr1) ===
            cr1 = nodes.new("ShaderNodeValToRGB")
            cr1.location = (-400, 0)
            cr1.color_ramp.elements[0].position = 0.3
            cr1.color_ramp.elements[1].position = 0.765
            cr1.color_ramp.elements[0].color = (0.329, 0.329, 0.329, 1)  # #545454
            cr1.color_ramp.elements[1].color = (0.804, 0.804, 0.804, 1)  # #CDCDCD
            links.new(noise.outputs["Fac"], cr1.inputs["Fac"])
            links.new(cr1.outputs["Color"], bsdf.inputs["Base Color"])
            
            # === Noise Texture 2 (nt2) ===
            nt2 = nodes.new("ShaderNodeTexNoise")
            nt2.location = (-600, -200)
            nt2.inputs["Scale"].default_value = 8.0
            nt2.inputs["Detail"].default_value = 16
            nt2.inputs["Roughness"].default_value = 0.8
            links.new(texcoord.outputs["Object"], nt2.inputs["Vector"])
            
            # === ColorRamp 2 (cr2) ===
            cr2 = nodes.new("ShaderNodeValToRGB")
            cr2.location = (-400, -200)
            cr2.color_ramp.elements[0].color = (0.556, 0.556, 0.556, 1)  # Light grey
            cr2.color_ramp.elements[1].color = (0.441, 0.441, 0.441, 1)  # Dark grey
            cr2.color_ramp.elements[1].position = 0.755
            links.new(nt2.outputs["Fac"], cr2.inputs["Fac"])
            links.new(cr2.outputs["Color"], bsdf.inputs["Roughness"])
            
            # === Bump Node ===
            bump = nodes.new("ShaderNodeBump")
            bump.location = (-200, -400)
            bump.inputs["Strength"].default_value = 0.02
            links.new(cr2.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
        
            
        elif props.material_type == "METAL" and props.metal_type == "STEEL":
            spacing_x = 300
            spacing_y = 150
            # Base Nodes
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Base Color"].default_value = (180/255, 180/255, 180/255, 1)  # B4B4B4 gray
            bsdf.inputs["Metallic"].default_value = 1.0
            output = nodes.get("Material Output")
            # Texture Coordinate & Mapping
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 4, 0)
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 3, 0)
            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
            # Duplicate Noise Texture (nt2)
            nt2 = nodes.new(type="ShaderNodeTexNoise")
            nt2.location = (-spacing_x * 2, spacing_y * 1)
            nt2.inputs["Scale"].default_value = 4.0
            nt2.inputs["Detail"].default_value = 10.0
            nt2.inputs["Roughness"].default_value = 0.45
            # Mix Color ll2 (Voronoi + Mapping)
            ll2 = nodes.new(type="ShaderNodeMixRGB")
            ll2.location = (-spacing_x * 1, spacing_y * 2)
            ll2.blend_type = "LINEAR_LIGHT"
            ll2.inputs["Fac"].default_value = 0.3
            voronoi = nodes.new(type="ShaderNodeTexVoronoi")
            voronoi.location = (-spacing_x, spacing_y * 3)
            voronoi.inputs["Scale"].default_value = 20.0
            voronoi.inputs["Detail"].default_value = 10.0
            links.new(mapping.outputs["Vector"], voronoi.inputs["Vector"])
            links.new(mapping.outputs["Vector"], ll2.inputs["Color1"])
            links.new(voronoi.outputs["Distance"], ll2.inputs["Color2"])
            links.new(ll2.outputs["Color"], nt2.inputs["Vector"])
            # Original Noise Texture (nt)
            nt = nodes.new(type="ShaderNodeTexNoise")
            nt.location = (0, spacing_y * 1)
            nt.inputs["Scale"].default_value = 3.0
            nt.inputs["Detail"].default_value = 10.0
            nt.inputs["Roughness"].default_value = 1.0
            # Mix Color before noise (ll1)
            ll1 = nodes.new(type="ShaderNodeMixRGB")
            ll1.location = (-spacing_x, spacing_y * 1)
            ll1.blend_type = "LINEAR_LIGHT"
            ll1.inputs["Fac"].default_value = 0.03
            links.new(mapping.outputs["Vector"], ll1.inputs["Color1"])
            links.new(nt2.outputs["Color"], ll1.inputs["Color2"])
            links.new(ll1.outputs["Color"], nt.inputs["Vector"])
            # ColorRamp for Roughness
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (spacing_x, spacing_y * 1)
            color_ramp.color_ramp.elements[0].position = 0.390
            color_ramp.color_ramp.elements[1].position = 0.617
            color_ramp.color_ramp.elements[1].color = (170/255, 170/255, 170/255, 1)  # Hex AAAAAA
            links.new(nt.outputs["Color"], color_ramp.inputs["Fac"])
            links.new(color_ramp.outputs["Color"], bsdf.inputs["Roughness"])
            # Bump Mapping
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x, -spacing_y * 1)
            bump.inputs["Strength"].default_value = 0.01
            links.new(nt.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            # Final Output
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
            
        


            
        elif props.material_type == "METAL" and props.metal_type == "DARK_STEEL":
            spacing_x = 300
            spacing_y = 150
            # Base Nodes
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Base Color"].default_value = (0.07, 0.07, 0.07, 1)  # Dark steel color
            bsdf.inputs["Metallic"].default_value = 1.0
            output = nodes.get("Material Output")
            # Texture Coordinate & Mapping
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 4, 0)
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 3, 0)
            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
            # Duplicate Noise Texture (nt2)
            nt2 = nodes.new(type="ShaderNodeTexNoise")
            nt2.location = (-spacing_x * 2, spacing_y * 1)
            nt2.inputs["Scale"].default_value = 4.0
            nt2.inputs["Detail"].default_value = 10.0
            nt2.inputs["Roughness"].default_value = 0.45
            # Mix Color ll2 (Voronoi + Mapping)
            ll2 = nodes.new(type="ShaderNodeMixRGB")
            ll2.location = (-spacing_x * 1, spacing_y * 2)
            ll2.blend_type = "LINEAR_LIGHT"
            ll2.inputs["Fac"].default_value = 0.3
            voronoi = nodes.new(type="ShaderNodeTexVoronoi")
            voronoi.location = (-spacing_x, spacing_y * 3)
            voronoi.inputs["Scale"].default_value = 20.0
            voronoi.inputs["Detail"].default_value = 10.0
            links.new(mapping.outputs["Vector"], voronoi.inputs["Vector"])
            links.new(mapping.outputs["Vector"], ll2.inputs["Color1"])
            links.new(voronoi.outputs["Distance"], ll2.inputs["Color2"])
            links.new(ll2.outputs["Color"], nt2.inputs["Vector"])
            # Original Noise Texture (nt)
            nt = nodes.new(type="ShaderNodeTexNoise")
            nt.location = (0, spacing_y * 1)
            nt.inputs["Scale"].default_value = 3.0
            nt.inputs["Detail"].default_value = 10.0
            nt.inputs["Roughness"].default_value = 1.0
            # Mix Color before noise (ll1)
            ll1 = nodes.new(type="ShaderNodeMixRGB")
            ll1.location = (-spacing_x, spacing_y * 1)
            ll1.blend_type = "LINEAR_LIGHT"
            ll1.inputs["Fac"].default_value = 0.03
            links.new(mapping.outputs["Vector"], ll1.inputs["Color1"])
            links.new(nt2.outputs["Color"], ll1.inputs["Color2"])
            links.new(ll1.outputs["Color"], nt.inputs["Vector"])
            # ColorRamp for Roughness
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (spacing_x, spacing_y * 1)
            color_ramp.color_ramp.elements[0].position = 0.390
            color_ramp.color_ramp.elements[1].position = 0.617
            color_ramp.color_ramp.elements[1].color = (170/255, 170/255, 170/255, 1)  # Hex AAAAAA
            links.new(nt.outputs["Color"], color_ramp.inputs["Fac"])
            links.new(color_ramp.outputs["Color"], bsdf.inputs["Roughness"])
            # Bump Mapping
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x, -spacing_y * 1)
            bump.inputs["Strength"].default_value = 0.01
            links.new(nt.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            # Final Output
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
        
        elif props.material_type == "METAL" and props.metal_type == "DARK_SHINY":
            spacing_x = 300
            spacing_y = 150
            
            # Principled BSDF
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Base Color"].default_value = (0.07, 0.07, 0.07, 1)
            bsdf.inputs["Metallic"].default_value = 1.0
            bsdf.inputs["Roughness"].default_value = 0.08  # Very shiny
            
            # Bump Node for subtle surface texture
            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-spacing_x * 2, 0)
            noise.inputs["Scale"].default_value = 15
            noise.inputs["Detail"].default_value = 8
            noise.inputs["Roughness"].default_value = 0.6
            
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (-spacing_x, 0)
            bump.inputs["Strength"].default_value = 0.03
            
            links.new(noise.outputs["Fac"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            
            # Output
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
       
       




        elif props.material_type == "PLASTIC":
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Metallic"].default_value = 0.0
            bsdf.inputs["Base Color"].default_value = props.base_color
            
            # ✅ Apply user-defined roughness value
            bsdf.inputs["Roughness"].default_value = props.plastic_roughness  # 🔥 Makes Roughness adjustable
            
            # ✅ Keep existing noise texture and bump details
            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-600, 0)
            noise.inputs["Scale"].default_value = 40.0
            noise.inputs["Detail"].default_value = 10.0
            noise.inputs["Roughness"].default_value = 0.84
            
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (-300, 0)
            bump.inputs["Strength"].default_value = 0.042
            links.new(noise.outputs["Fac"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])

       
            
        elif props.material_type == "RUBBER" and props.rubber_type == "NORMAL"  and props.rubber_type != "":
            # Define spacing for nodes
            spacing_x = 300  # Horizontal spacing
            spacing_y = 150  # Vertical spacing
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Metallic"].default_value = 0.0  # No metallic properties
            bsdf.inputs["Roughness"].default_value = 0.6  # Matte rubber texture
            bsdf.inputs["IOR"].default_value = 1.450  # Index of Refraction for rubber
            bsdf.inputs["Base Color"].default_value = (0.02, 0.02, 0.02, 1)  # Dark gray
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])  # ✅ Connect BSDF to Output

            
            output = nodes.new(type="ShaderNodeOutputMaterial")
            output.location = (spacing_x * 2, 0)
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])
            
            
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x, -spacing_y)
            bump.inputs["Strength"].default_value = 0.160
            
            
            math_node = nodes.new(type="ShaderNodeMath")
            math_node.location = (spacing_x, -spacing_y * 2)
            math_node.operation = 'MULTIPLY'
            math_node.inputs[1].default_value = -1  # ✅ Invert effect
            links.new(bump.inputs["Height"], math_node.outputs["Value"])

            
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (spacing_x, spacing_y)
            color_ramp.color_ramp.elements[0].position = 0.514  # Black slider position
            color_ramp.color_ramp.elements[1].position = 1.0  # White slider at same place
            links.new(color_ramp.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            
            noise_texture = nodes.new(type="ShaderNodeTexNoise")
            noise_texture.location = (-spacing_x, spacing_y)
            noise_texture.inputs["Scale"].default_value = 150
            noise_texture.inputs["Detail"].default_value = 5
            noise_texture.inputs["Roughness"].default_value = 0.555
            noise_texture.inputs["Distortion"].default_value = 2.0
            links.new(noise_texture.outputs["Fac"], color_ramp.inputs["Fac"])
            
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 2, spacing_y)
            links.new(tex_coord.outputs["Object"], noise_texture.inputs["Vector"])
           
        elif props.material_type == "RUBBER":
            # ✅ Create Principled BSDF (Base Material)
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Base Color"].default_value = props.base_color  # 🎨 User-selected color
            bsdf.inputs["Metallic"].default_value = 0.0  # Rubber is non-metallic
            bsdf.inputs["Roughness"].default_value = 0.9  # Matte rubber surface
            bsdf.inputs["IOR"].default_value = 1.45  # Rubber Index of Refraction
            
            # ✅ Create Texture Coordinate Node
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-800, 0)
            
            # ✅ Create Mapping Node for Noise Texture
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-600, 0)
            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])  # ✅ Connect Object to Mapping
            
            # ✅ Create Noise Texture (for rubber details)
            noise_texture = nodes.new(type="ShaderNodeTexNoise")
            noise_texture.location = (-400, 0)
            noise_texture.inputs["Scale"].default_value = 300  # 🔥 High scale for fine details
            noise_texture.inputs["Detail"].default_value = 7  # Moderate detail level
            noise_texture.inputs["Roughness"].default_value = 0.6  # Slight roughness variation
            links.new(mapping.outputs["Vector"], noise_texture.inputs["Vector"])  # ✅ Connect Mapping to Noise Texture
            
            # ✅ Create Bump Node for surface texture
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (-200, 0)
            bump.inputs["Strength"].default_value = 0.3  # 🔥 Visible surface texture
            links.new(noise_texture.outputs["Fac"], bump.inputs["Height"])  # ✅ Noise to Bump Height
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])  # ✅ Bump Normal to BSDF Normal
            
            # ✅ Final Output
            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])


        elif props.material_type == "WOOD" and props.wood_type == "DEFAULT":
            spacing_x = 300  # Horizontal spacing
            spacing_y = 150  # Vertical spacing
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)

            bsdf.inputs["Metallic"].default_value = 0.0
            bsdf.inputs["Roughness"].default_value = 0.505

            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 3, spacing_y * 2)
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 2, spacing_y * 2)
            mapping.inputs['Scale'].default_value = (1.6, 5.7, 1.0)

            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-spacing_x, spacing_y * 2)
            noise.inputs["Scale"].default_value = 1.9
            noise.inputs["Detail"].default_value = 10
            noise.inputs["Roughness"].default_value = 1
            noise.inputs["Distortion"].default_value = 1

            color_ramp_color = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_color.location = (0, spacing_y * 2)
            dark_hex = (117/255, 50/255, 25/255, 1)
            light_hex = (21/255, 10/255, 4/255, 1)
            color_ramp_color.color_ramp.elements[0].position = 0.38
            color_ramp_color.color_ramp.elements[1].position = 0.62
            color_ramp_color.color_ramp.elements[0].color = dark_hex
            color_ramp_color.color_ramp.elements[1].color = light_hex

            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x, spacing_y * 1)
            bump.inputs["Strength"].default_value = 0.01
            color_ramp_bump = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_bump.location = (-300, -200)

            disp = nodes.new(type="ShaderNodeDisplacement")
            disp.location = (spacing_x * 2, spacing_y * 1)
            disp.inputs["Scale"].default_value = 0.02
            color_ramp_disp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_disp.location = (spacing_x * 2, spacing_y * 2)
            
            links.new(bsdf.outputs['BSDF'], output.inputs['Surface'])
            links.new(tex_coord.outputs["Generated"], mapping.inputs["Vector"])
            links.new(mapping.outputs["Vector"], noise.inputs["Vector"])
            links.new(noise.outputs["Fac"], color_ramp_color.inputs["Fac"])
            links.new(color_ramp_color.outputs["Color"], bsdf.inputs["Base Color"])
            links.new(noise.outputs["Fac"], color_ramp_bump.inputs["Fac"])
            links.new(color_ramp_bump.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            links.new(noise.outputs["Fac"], color_ramp_disp.inputs["Fac"])
            links.new(color_ramp_disp.outputs["Color"], disp.inputs["Height"])
            links.new(disp.outputs["Displacement"], output.inputs["Displacement"])
            mat.cycles.displacement_method = 'DISPLACEMENT'
        
        elif props.material_type == "WOOD" and props.wood_type == "DARK":
            spacing_x = 300  # Horizontal spacing
            spacing_y = 150  # Vertical spacing
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)

            bsdf.inputs["Metallic"].default_value = 0.0
            bsdf.inputs["Roughness"].default_value = 0.550

            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 3, spacing_y * 2)
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 2, spacing_y * 2)
            mapping.inputs['Scale'].default_value = (1.6, 5.7, 1.0)

            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-spacing_x, spacing_y * 2)
            noise.inputs["Scale"].default_value = 1.9
            noise.inputs["Detail"].default_value = 10
            noise.inputs["Roughness"].default_value = 1
            noise.inputs["Distortion"].default_value = 1

            color_ramp_color = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_color.location = (0, spacing_y * 2)
            dark_hex = (8/255, 3/255, 2/255, 1) # Rich reddish-brown
            light_hex = (110/255, 10/255, 5/255, 1) # Slightly lighter red-brown
            color_ramp_color.color_ramp.elements[0].position = 0.38
            color_ramp_color.color_ramp.elements[1].position = 0.62
            color_ramp_color.color_ramp.elements[0].color = dark_hex
            color_ramp_color.color_ramp.elements[1].color = light_hex

            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x, spacing_y * 1)
            bump.inputs["Strength"].default_value = 0.01
            color_ramp_bump = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_bump.location = (spacing_x, spacing_y * 2)

            disp = nodes.new(type="ShaderNodeDisplacement")
            disp.location = (spacing_x * 2, spacing_y * 1)
            disp.inputs["Scale"].default_value = 0.02
            color_ramp_disp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_disp.location = (-200, -300)
            
            links.new(bsdf.outputs['BSDF'], output.inputs['Surface'])
            links.new(tex_coord.outputs["Generated"], mapping.inputs["Vector"])
            links.new(mapping.outputs["Vector"], noise.inputs["Vector"])
            links.new(noise.outputs["Fac"], color_ramp_color.inputs["Fac"])
            links.new(color_ramp_color.outputs["Color"], bsdf.inputs["Base Color"])
            links.new(noise.outputs["Fac"], color_ramp_bump.inputs["Fac"])
            links.new(color_ramp_bump.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            links.new(noise.outputs["Fac"], color_ramp_disp.inputs["Fac"])
            links.new(color_ramp_disp.outputs["Color"], disp.inputs["Height"])
            links.new(disp.outputs["Displacement"], output.inputs["Displacement"])
            mat.cycles.displacement_method = 'DISPLACEMENT'
            
        elif props.material_type == "WOOD" and props.wood_type == "LIGHT":
            spacing_x = 300  # Horizontal spacing
            spacing_y = 150  # Vertical spacing
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)

            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 3, spacing_y * 2)
            bsdf.inputs["Metallic"].default_value = 0.0
            bsdf.inputs["Roughness"].default_value = 0.65
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 3, spacing_y * 2)
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 2, spacing_y * 2)
            mapping.inputs['Scale'].default_value = (1.6, 5.7, 1.0)

            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-spacing_x, spacing_y * 2)
            noise.inputs["Scale"].default_value = 1.9
            noise.inputs["Detail"].default_value = 10
            noise.inputs["Roughness"].default_value = 1
            noise.inputs["Distortion"].default_value = 1
            
            color_ramp_color = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_color.location = (0, spacing_y * 2)
            light_hex = (175/255, 115/255, 45/255, 1)  # Pale beige
            mid_hex = (85/255, 50/255, 20/255, 1)    # Soft tan
            color_ramp_color.color_ramp.elements[0].position = 0.38
            color_ramp_color.color_ramp.elements[1].position = 0.62
            color_ramp_color.color_ramp.elements[0].color = light_hex
            color_ramp_color.color_ramp.elements[1].color = mid_hex

            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x, spacing_y * 1)
            bump.inputs["Strength"].default_value = 0.01
            color_ramp_bump = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_bump.location = (spacing_x, spacing_y * 2)

            disp = nodes.new(type="ShaderNodeDisplacement")
            disp.location = (spacing_x * 2, spacing_y * 1)
            disp.inputs["Scale"].default_value = 0.02
            color_ramp_disp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp_disp.location = (spacing_x * 2, spacing_y * 2)
            
            links.new(bsdf.outputs['BSDF'], output.inputs['Surface'])
            links.new(tex_coord.outputs["Generated"], mapping.inputs["Vector"])
            links.new(mapping.outputs["Vector"], noise.inputs["Vector"])
            links.new(noise.outputs["Fac"], color_ramp_color.inputs["Fac"])
            links.new(color_ramp_color.outputs["Color"], bsdf.inputs["Base Color"])
            links.new(noise.outputs["Fac"], color_ramp_bump.inputs["Fac"])
            links.new(color_ramp_bump.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])
            links.new(noise.outputs["Fac"], color_ramp_disp.inputs["Fac"])
            links.new(color_ramp_disp.outputs["Color"], disp.inputs["Height"])
            links.new(disp.outputs["Displacement"], output.inputs["Displacement"])
            mat.cycles.displacement_method = 'DISPLACEMENT'
            
        elif props.material_type == "GLASS":
            spacing_x = 280
            spacing_y = 160

            # Output and BSDF
            bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            bsdf.location = (0, 0)
            bsdf.inputs["Base Color"].default_value = props.glass_base_color
            bsdf.inputs["Transmission Weight"].default_value = 1.0


            links.new(bsdf.outputs["BSDF"], output.inputs["Surface"])

            # Texture Coordinate and Mapping
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-spacing_x * 5, spacing_y)

            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-spacing_x * 4, spacing_y)
            mapping.inputs["Scale"].default_value = (
                props.glass_texture_scale,
                props.glass_texture_scale,
                props.glass_texture_scale
            )

            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])

            # --- Smudges Imperfection ---
            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-spacing_x * 3, spacing_y * 2)
            noise.inputs["Scale"].default_value = 16
            noise.inputs["Detail"].default_value = 15
            noise.inputs["Roughness"].default_value = 1.0
            links.new(mapping.outputs["Vector"], noise.inputs["Vector"])

            mix = nodes.new(type="ShaderNodeMixRGB")
            mix.blend_type = 'LINEAR_LIGHT'
            mix.inputs["Fac"].default_value = 0.14
            mix.location = (-spacing_x * 2, spacing_y * 2)
            links.new(mapping.outputs["Vector"], mix.inputs[1])
            links.new(noise.outputs["Fac"], mix.inputs[2])

            voronoi = nodes.new(type="ShaderNodeTexVoronoi")
            voronoi.location = (-spacing_x, spacing_y * 2)
            voronoi.feature = 'SMOOTH_F1'
            voronoi.inputs["Scale"].default_value = 18
            voronoi.inputs["Detail"].default_value = 15
            links.new(mix.outputs["Color"], voronoi.inputs["Vector"])

            cr_smudges = nodes.new(type="ShaderNodeValToRGB")
            cr_smudges.location = (0, spacing_y * 2)
            cr_smudges.color_ramp.elements[0].position = 0.580
            cr_smudges.color_ramp.elements[1].position = 0.935
            cr_smudges.color_ramp.elements[0].color = (1, 1, 1, 1)
            cr_smudges.color_ramp.elements[1].color = (0, 0, 0, 1)
            links.new(voronoi.outputs["Distance"], cr_smudges.inputs["Fac"])

            # --- Noise Roughness Imperfection ---
            nt2 = nodes.new(type="ShaderNodeTexNoise")
            nt2.location = (-spacing_x * 3, 0)
            nt2.inputs["Scale"].default_value = 30
            nt2.inputs["Detail"].default_value = 15
            nt2.inputs["Roughness"].default_value = 0.8
            links.new(mapping.outputs["Vector"], nt2.inputs["Vector"])

            cr2 = nodes.new(type="ShaderNodeValToRGB")
            cr2.location = (-spacing_x * 2, 0)
            cr2.color_ramp.elements[0].position = 0.302
            links.new(nt2.outputs["Fac"], cr2.inputs["Fac"])

            mix2 = nodes.new(type="ShaderNodeMixRGB")
            mix2.blend_type = 'MIX'
            mix2.location = (-spacing_x, 0)
            mix2.inputs["Fac"].default_value = props.glass_roughness
            mix2.inputs["Color1"].default_value = (0, 0, 0, 1)
            links.new(cr2.outputs["Color"], mix2.inputs["Color2"])

            bc = nodes.new(type="ShaderNodeBrightContrast")
            bc.location = (0, 0)
            

            links.new(mix2.outputs["Color"], bc.inputs["Color"])

            # --- Crack Imperfection ---
            map2 = nodes.new(type="ShaderNodeMapping")
            map2.location = (-spacing_x * 4, -spacing_y * 2)
            map2.inputs["Scale"].default_value = (
                props.glass_crack_scale,
                props.glass_crack_scale,
                props.glass_crack_scale
            )

            links.new(mapping.outputs["Vector"], map2.inputs["Vector"])

            nt3 = nodes.new(type="ShaderNodeTexNoise")
            nt3.location = (-spacing_x * 3, -spacing_y * 2)
            nt3.inputs["Scale"].default_value = 20
            nt3.inputs["Detail"].default_value = 15
            nt3.inputs["Roughness"].default_value = 0.6
            links.new(map2.outputs["Vector"], nt3.inputs["Vector"])

            v2 = nodes.new(type="ShaderNodeTexVoronoi")
            v2.location = (-spacing_x * 2, -spacing_y * 2)
            v2.feature = 'DISTANCE_TO_EDGE'
            v2.inputs["Scale"].default_value = 15
            v2.inputs["Lacunarity"].default_value = 3
            v2.inputs["Roughness"].default_value = 0.55

            v3 = nodes.new(type="ShaderNodeTexVoronoi")
            v3.location = (-spacing_x * 2, -spacing_y * 3)
            v3.inputs["Scale"].default_value = 30

            mix4 = nodes.new(type="ShaderNodeMixRGB")
            mix4.blend_type = 'LINEAR_LIGHT'
            mix4.location = (-spacing_x, -spacing_y * 3)
            mix4.inputs["Fac"].default_value = 0.01
            links.new(map2.outputs["Vector"], mix4.inputs["Color1"])
            links.new(nt3.outputs["Color"], mix4.inputs["Color2"])
            links.new(mix4.outputs["Color"], v2.inputs["Vector"])
            links.new(mix4.outputs["Color"], v3.inputs["Vector"])

            mix3 = nodes.new(type="ShaderNodeMixRGB")
            mix3.blend_type = 'DARKEN'
            mix3.location = (0, -spacing_y * 3)
            mix3.inputs["Fac"].default_value = 1
            links.new(v2.outputs["Distance"], mix3.inputs["Color1"])
            links.new(v3.outputs["Distance"], mix3.inputs["Color2"])

            cr3 = nodes.new(type="ShaderNodeValToRGB")
            cr3.location = (spacing_x, -spacing_y * 3)
            cr3.color_ramp.elements[1].position = 0.005
            links.new(mix3.outputs["Color"], cr3.inputs["Fac"])

            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (spacing_x * 1.5, -spacing_y * 2)
            bump.inputs["Strength"].default_value = props.glass_crack_strength
            links.new(cr3.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])

            # --- Combine Imperfections into BSDF ---
            mix5 = nodes.new(type="ShaderNodeMixRGB")
            mix5.blend_type = 'ADD'
            mix5.inputs["Fac"].default_value = props.smudge_strength
            mix5.location = (spacing_x, spacing_y)
            links.new(cr_smudges.outputs["Color"], mix5.inputs["Color2"])
            links.new(bc.outputs["Color"], mix5.inputs["Color1"])
            links.new(mix5.outputs["Color"], bsdf.inputs["Roughness"])      
            
        # ✅ Apply Edge Wear Effect (only for Plastic or Metal (NONE))
        if props.imperfections == "EDGE_WEAR" and (props.material_type == "PLASTIC" or (props.material_type == "METAL" and props.metal_type == "NONE")):
            
            spacing_x, spacing_y = 300, 150
            frame = nodes.new("NodeFrame")
            frame.label = "Edge Wear Imperfection"
            frame.name = "Edge Wear Imperfection"
            # Mix Shader
            mix_shader = nodes.new(type="ShaderNodeMixShader")
            mix_shader.location = (300, 0)
            
            # Bevel
            bevel = nodes.new(type="ShaderNodeBevel")
            bevel.location = (-600, 0)
            bevel.samples = 16
            bevel.inputs["Radius"].default_value = 0.050
            bevel.parent = frame
            
            # Texture Coordinate
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-1000, -200)
            tex_coord.parent = frame
            
            # Vector Math (Dot Product)
            vector_math = nodes.new(type="ShaderNodeVectorMath")
            vector_math.operation = 'DOT_PRODUCT'
            vector_math.location = (-800, 0)
            links.new(tex_coord.outputs["Normal"], vector_math.inputs[0])
            links.new(bevel.outputs["Normal"], vector_math.inputs[1])
            vector_math.parent = frame
            
            # Map Range
            map_range = nodes.new(type="ShaderNodeMapRange")
            map_range.location = (-600, 200)
            map_range.inputs["From Min"].default_value = props.edge_wear_strength
            map_range.inputs["To Min"].default_value = 15.0
            map_range.inputs["To Max"].default_value = 0.0
            links.new(vector_math.outputs["Value"], map_range.inputs["Value"])
            map_range.parent = frame
            
            # Noise Texture
            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-800, -200)
            noise.inputs["Detail"].default_value = 15.0
            noise.inputs["Roughness"].default_value = 0.7
            noise.parent = frame
            
            # ColorRamp
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (-600, -200)
            color_ramp.color_ramp.elements[0].position = 0.523
            color_ramp.color_ramp.elements[1].position = 0.750

            links.new(noise.outputs["Fac"], color_ramp.inputs["Fac"])
            color_ramp.parent = frame
            
            # Math Add Node for final radius
            math_add = nodes.new(type="ShaderNodeMath")
            math_add.operation = "ADD"
            math_add.location = (-400, -200)
            links.new(color_ramp.outputs["Color"], math_add.inputs[0])
            math_add.inputs[1].default_value = props.edge_wear_spread
            links.new(math_add.outputs[0], bevel.inputs["Radius"])
            math_add.parent = frame
            
            # Edge Wear BSDF
            worn_bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            worn_bsdf.location = (0, 200)
            worn_bsdf.inputs["Base Color"].default_value = props.edge_wear_color
            worn_bsdf.inputs["Metallic"].default_value = 1.0
            worn_bsdf.inputs["Roughness"].default_value = 0.398
            links.new(bsdf.inputs["Normal"].links[0].from_node.outputs["Normal"], worn_bsdf.inputs["Normal"])
            worn_bsdf.parent = frame

            
            # Link edge wear to Mix Shader (as Shader 2 now)
            links.new(bsdf.outputs["BSDF"], mix_shader.inputs[1])           # User BSDF now Shader 1
            links.new(worn_bsdf.outputs["BSDF"], mix_shader.inputs[2])      # Edge Wear now Shader 2
            links.new(map_range.outputs["Result"], mix_shader.inputs["Fac"])
            
            # Final Output
            links.new(mix_shader.outputs["Shader"], output.inputs["Surface"])
            
            
            
        elif props.imperfections == "RUST" and (props.material_type == "METAL" and props.metal_type == "NONE"):

            spacing_x, spacing_y = 300, 150
            frame = nodes.new("NodeFrame")
            frame.label = "Rust Imperfection"
            frame.name = "Rust Imperfection"
            # Mix Shader
            mix_shader = nodes.new(type="ShaderNodeMixShader")
            mix_shader.location = (300, 0)
            mix_shader.parent = frame
            
            # Rust BSDF
            rust_bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            rust_bsdf.location = (300, 0)
            rust_bsdf.inputs["Metallic"].default_value = 1.0
            rust_bsdf.parent = frame
            
            # ───── Edge wear mask setup ─────
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-1500, -400)
            tex_coord.parent = frame
            
            bevel = nodes.new(type="ShaderNodeBevel")
            bevel.location = (-1000, -200)
            bevel.samples = 16
            bevel.parent = frame
            
            vector_math = nodes.new(type="ShaderNodeVectorMath")
            vector_math.operation = 'DOT_PRODUCT'
            vector_math.location = (-800, -200)
            links.new(tex_coord.outputs["Normal"], vector_math.inputs[0])
            links.new(bevel.outputs["Normal"], vector_math.inputs[1])
            vector_math.parent = frame
            
            map_range = nodes.new(type="ShaderNodeMapRange")
            map_range.location = (-600, -200)
            map_range.inputs["From Min"].default_value = 0.960
            map_range.inputs["To Min"].default_value = 15.0
            map_range.inputs["To Max"].default_value = 0.0
            links.new(vector_math.outputs["Value"], map_range.inputs["Value"])
            map_range.parent = frame
            
            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-1000, -400)
            noise.inputs["Detail"].default_value = 10.0
            noise.inputs["Roughness"].default_value = 0.89
            noise.inputs["Distortion"].default_value = 0.3
            links.new(tex_coord.outputs["Object"], noise.inputs["Vector"])
            noise.parent = frame
            
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (-800, -400)
            color_ramp.color_ramp.elements[0].position = 0.523
            color_ramp.color_ramp.elements[1].position = 0.750
            links.new(noise.outputs["Fac"], color_ramp.inputs["Fac"])
            color_ramp.parent = frame
            
            math_add = nodes.new(type="ShaderNodeMath")
            math_add.operation = "ADD"
            math_add.location = (-400, -400)
            links.new(color_ramp.outputs["Color"], math_add.inputs[0])
            math_add.inputs[1].default_value = props.rust_spread
            links.new(math_add.outputs[0], bevel.inputs["Radius"])
            math_add.parent = frame
            
            # ───── Procedural rust material setup ─────
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-1300, 0)
            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
            mapping.parent = frame
            
            # nt1
            nt1 = nodes.new(type="ShaderNodeTexNoise")
            nt1.location = (-1100, 200)
            nt1.inputs["Scale"].default_value = 3.2
            nt1.inputs["Detail"].default_value = 15
            nt1.inputs["Roughness"].default_value = 0.65
            links.new(mapping.outputs["Vector"], nt1.inputs["Vector"])
            nt1.parent = frame
            
            # cr1
            cr1 = nodes.new(type="ShaderNodeValToRGB")
            cr1.location = (-900, 200)
            cr1.color_ramp.elements[0].position = 0.538
            cr1.color_ramp.elements[0].color = (0.4, 0.21, 0.02, 1)
            mid = cr1.color_ramp.elements.new(0.678)
            mid.color = (0.38, 0.38, 0.37, 1)
            cr1.color_ramp.elements[1].position = 0.791
            cr1.color_ramp.elements[1].color = (0.009, 0.007, 0.002, 1)
            links.new(nt1.outputs["Fac"], cr1.inputs["Fac"])
            cr1.parent = frame
            
            # nt2
            nt2 = nodes.new(type="ShaderNodeTexNoise")
            nt2.location = (-1100, -100)
            nt2.inputs["Scale"].default_value = 7.0
            nt2.inputs["Detail"].default_value = 15
            nt2.inputs["Roughness"].default_value = 0.65
            links.new(mapping.outputs["Vector"], nt2.inputs["Vector"])
            nt2.parent = frame
            
            # cr2
            cr2 = nodes.new(type="ShaderNodeValToRGB")
            cr2.location = (-900, -100)
            cr2.color_ramp.elements[0].position = 0.455
            cr2.color_ramp.elements[1].position = 0.582
            links.new(nt2.outputs["Fac"], cr2.inputs["Fac"])
            cr2.parent = frame
            
            # mix1 (base color)
            mix1 = nodes.new(type="ShaderNodeMixRGB")
            mix1.location = (-600, 200)
            mix1.blend_type = 'MIX'
            mix1.inputs["Fac"].default_value = 1.0
            mix1.inputs["Color2"].default_value = (0.584, 0.282, 0.0, 1)
            links.new(cr1.outputs["Color"], mix1.inputs["Color1"])
            links.new(cr2.outputs["Color"], mix1.inputs["Fac"])
            mix1.parent = frame
            
            # mix2 (roughness)
            mix2 = nodes.new(type="ShaderNodeMixRGB")
            mix2.location = (-600, 0)
            mix2.blend_type = 'ADD'
            mix2.inputs["Color1"].default_value = (0.71, 0.71, 0.71, 1)
            links.new(cr1.outputs["Color"], mix2.inputs["Color2"])
            links.new(cr2.outputs["Color"], mix2.inputs["Fac"])
            mix2.parent = frame
            
            # Bump nodes
            bump1 = nodes.new(type="ShaderNodeBump")
            bump1.location = (-300, 200)
            bump1.inputs["Strength"].default_value = 0.14
            links.new(cr1.outputs["Color"], bump1.inputs["Height"])
            bump1.parent = frame
            
            bump3 = nodes.new(type="ShaderNodeBump")
            bump3.location = (-100, 200)
            bump3.inputs["Strength"].default_value = 0.1
            links.new(nt2.outputs["Fac"], bump3.inputs["Height"])
            links.new(bump1.outputs["Normal"], bump3.inputs["Normal"])
            bump3.parent = frame
            
            bump2 = nodes.new(type="ShaderNodeBump")
            bump2.location = (100, 200)
            bump2.inputs["Strength"].default_value = 0.31
            bump2.invert = True
            links.new(cr2.outputs["Color"], bump2.inputs["Height"])
            links.new(bump3.outputs["Normal"], bump2.inputs["Normal"])
            bump2.parent = frame
            
            darkness_factor = max(0.0, min(1.0, 1.0 - props.rust_darkness))

            for elem in cr1.color_ramp.elements:
                r, g, b, a = elem.color
                elem.color = (r * darkness_factor, g * darkness_factor, b * darkness_factor, a)

            r, g, b, a = mix1.inputs["Color2"].default_value
            mix1.inputs["Color2"].default_value = (r * darkness_factor, g * darkness_factor, b * darkness_factor, a)

            
            # Final rust BSDF wiring
            links.new(mix1.outputs["Color"], rust_bsdf.inputs["Base Color"])
            links.new(nt1.outputs["Fac"], rust_bsdf.inputs["Metallic"])
            links.new(mix2.outputs["Color"], rust_bsdf.inputs["Roughness"])
            links.new(bump2.outputs["Normal"], rust_bsdf.inputs["Normal"])
            
            # Mix with user BSDF using rust mask
            links.new(bsdf.outputs["BSDF"], mix_shader.inputs[1])
            links.new(rust_bsdf.outputs["BSDF"], mix_shader.inputs[2])
            links.new(map_range.outputs["Result"], mix_shader.inputs["Fac"])
            
            # Final output
            links.new(mix_shader.outputs["Shader"], output.inputs["Surface"])
            
            
        elif props.imperfections == "DOWN_DIRT" and ( props.material_type == "PLASTIC" or (props.material_type == "METAL" and props.metal_type == "NONE")):
            frame = nodes.new("NodeFrame")
            frame.label = "Down Dirt Imperfection"
            frame.name = "Down Dirt Imperfection"
            # TEXTURE COORDINATE
            tex_coord = nodes.new("ShaderNodeTexCoord")
            tex_coord.location = (-1600, 0)
            tex_coord.parent = frame

            # SEPARATE XYZ
            separate_xyz = nodes.new("ShaderNodeSeparateXYZ")
            separate_xyz.location = (-1400, 0)
            links.new(tex_coord.outputs["Generated"], separate_xyz.inputs["Vector"])
            separate_xyz.parent = frame

            # MATH NODE 2 (Multiply)
            math2 = nodes.new("ShaderNodeMath")
            math2.location = (-1200, -150)
            math2.operation = "MULTIPLY"
            math2.inputs[1].default_value = props.dirt_spread
            math2.parent = frame


            # VECTOR MATH
            vector_math = nodes.new("ShaderNodeVectorMath")
            vector_math.location = (-1200, 100)
            vector_math.operation = "ADD"
            links.new(separate_xyz.outputs["Z"], vector_math.inputs[0])  # Z to vector math (X input)
            vector_math.parent = frame

            # NOISE TEXTURE (4D)
            noise_mask = nodes.new("ShaderNodeTexNoise")
            noise_mask.location = (-1400, -200)
            noise_mask.noise_dimensions = '4D'
            noise_mask.inputs["W"].default_value = 8.6
            noise_mask.inputs["Scale"].default_value = 1.5
            noise_mask.inputs["Detail"].default_value = 15
            links.new(tex_coord.outputs["Object"], noise_mask.inputs["Vector"])
            links.new(noise_mask.outputs["Color"], vector_math.inputs[1])  # Color to second vector
            noise_mask.parent = frame

            # MATH NODE 1 (Add)
            math1 = nodes.new("ShaderNodeMath")
            math1.location = (-900, 0)
            math1.operation = "ADD"
            links.new(vector_math.outputs["Vector"], math1.inputs[0])
            links.new(math2.outputs["Value"], math1.inputs[1])
            math1.parent = frame

            # COLORRAMP (final dirt mask)
            dirt_mask_ramp = nodes.new("ShaderNodeValToRGB")
            dirt_mask_ramp.location = (-600, 0)
            brightness = 1.0 - props.dirt_opacity
            dirt_mask_ramp.color_ramp.elements[0].color = (brightness, brightness, brightness, 1.0)
            dirt_mask_ramp.parent = frame


            dirt_mask_ramp.color_ramp.elements[0].position = 0.009
            dirt_mask_ramp.color_ramp.elements[1].position = 0.280
            links.new(math1.outputs["Value"], dirt_mask_ramp.inputs["Fac"])
            dirt_noise.parent = frame

            # DIRT NOISE
            dirt_noise = nodes.new("ShaderNodeTexNoise")
            dirt_noise.location = (-400, -300)
            dirt_noise.inputs["Scale"].default_value = 5
            dirt_noise.inputs["Detail"].default_value = 10
            dirt_noise.inputs["Roughness"].default_value = 0.75
            links.new(tex_coord.outputs["Object"], dirt_noise.inputs["Vector"])
            dirt_color_ramp.parent = frame

            # COLORRAMP for DIRT COLOR
            dirt_color_ramp = nodes.new("ShaderNodeValToRGB")
            dirt_color_ramp.location = (-200, -300)
            dirt_color_ramp.color_ramp.elements[0].position = 0.086
            dirt_color_ramp.color_ramp.elements[1].position = 0.755
            dirt_color_ramp.color_ramp.elements[0].color = (*props.dirt_color_1, 1.0)
            dirt_color_ramp.color_ramp.elements[1].color = (*props.dirt_color_2, 1.0)
            links.new(dirt_noise.outputs["Fac"], dirt_color_ramp.inputs["Fac"])
            dirt_bsdf.parent = frame

            # DIRT BSDF
            dirt_bsdf = nodes.new("ShaderNodeBsdfPrincipled")
            dirt_bsdf.location = (0, -100)
            dirt_bsdf.inputs["Roughness"].default_value = props.dirt_roughness
            dirt_bsdf.inputs["Metallic"].default_value = props.dirt_metallic
            links.new(dirt_color_ramp.outputs["Color"], dirt_bsdf.inputs["Base Color"])
            dirt_bump.parent = frame

            # BUMP for dirt
            dirt_bump = nodes.new("ShaderNodeBump")
            dirt_bump.location = (-50, -300)
            dirt_bump.inputs["Strength"].default_value = props.dirt_strength
            links.new(dirt_noise.outputs["Color"], dirt_bump.inputs["Height"])
            links.new(dirt_bump.outputs["Normal"], dirt_bsdf.inputs["Normal"])

            # MIX SHADER (imperfection blending)
            mix_shader = nodes.new("ShaderNodeMixShader")
            mix_shader.location = (300, 0)
            links.new(dirt_mask_ramp.outputs["Color"], mix_shader.inputs["Fac"])
            links.new(dirt_bsdf.outputs["BSDF"], mix_shader.inputs[1])
            links.new(bsdf.outputs["BSDF"], mix_shader.inputs[2])  # Original material
            links.new(mix_shader.outputs["Shader"], output.inputs["Surface"])  # ✅ Correct
            
            
            
            
            
        elif props.imperfections == "DIRT" and ( props.material_type == "PLASTIC" or (props.material_type == "METAL" and props.metal_type == "NONE") or (props.material_type == "SPECIAL_METAL" and props.special_metal_type == "CAMO")):
            
            frame = nodes.new("NodeFrame")
            frame.label = "Dirt Imperfection"
            frame.name = "Dirt Imperfection"

            # ✅ Remove Existing Noise Texture if Present
            for node in nodes:
                if node.type == "ShaderNodeTexNoise":
                    nodes.remove(node)  # 🔥 Deletes existing Noise Texture
            # ✅ Create Dirt Imperfection BSDF Shader
            dirt_bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            dirt_bsdf.location = (-300, 0)
            dirt_bsdf.inputs["Metallic"].default_value = 1.0  # Fully metallic dirt
            dirt_bsdf.inputs["Roughness"].default_value = props.dirt_roughness
            dirt_bsdf.inputs["Base Color"].default_value = props.dirt_color  # ✅ dynamic from N-panel
            dirt_bsdf.parent = frame
            
            tex_coord = nodes.new("ShaderNodeTexCoord")
            tex_coord.location = (-1000, 150)
            tex_coord.parent = frame
            
            mapping = nodes.new("ShaderNodeMapping")
            mapping.location = (-800, 150)
            mapping.inputs["Location"].default_value = props.dirt_mapping_location
            mapping.inputs["Scale"].default_value = (
            props.dirt_mapping_scale,
            props.dirt_mapping_scale,
            props.dirt_mapping_scale
            )


            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
            mapping.parent = frame


            
            # ✅ New Noise Texture for Dirt Mask
            noise_texture = nodes.new(type="ShaderNodeTexNoise")
            noise_texture.location = (-600, 150)
            noise_texture.inputs["Scale"].default_value = 3.5
            noise_texture.inputs["Detail"].default_value = 10
            noise_texture.inputs["Roughness"].default_value = 0.9
            links.new(mapping.outputs["Vector"], noise_texture.inputs["Vector"])
            noise_texture.parent = frame
            
            # ✅ ColorRamp Node for Dirt Mask Refinement
            color_ramp = nodes.new(type="ShaderNodeValToRGB")
            color_ramp.location = (-400, 150)
            color_ramp.color_ramp.elements[0].position = 0.545
            color_ramp.color_ramp.elements[1].position = 0.736
            
            links.new(noise_texture.outputs["Fac"], color_ramp.inputs["Fac"])
            color_ramp.parent = frame
            
            # ✅ Bump Node for Dirt Surface Detail
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (-200, -150)
            bump.inputs["Strength"].default_value = 0.1
            
            links.new(color_ramp.outputs["Color"], bump.inputs["Height"])
            links.new(bump.outputs["Normal"], dirt_bsdf.inputs["Normal"])  # 🎯 Connect Bump to Dirt BSDF!
            bump.parent = frame
            
            # ✅ Mix Shader Node to Blend Dirt with Main Material
            mix_shader = nodes.new(type="ShaderNodeMixShader")
            mix_shader.location = (200, 0)
            
            links.new(bsdf.outputs["BSDF"], mix_shader.inputs[1])  # Base Material
            links.new(dirt_bsdf.outputs["BSDF"], mix_shader.inputs[2])  # Dirt Shader
            links.new(color_ramp.outputs["Color"], mix_shader.inputs[0])  # Mask as Mix Factor
            
            links.new(mix_shader.outputs["Shader"], output.inputs["Surface"])  # Final Output
            
        elif props.imperfections == "SCRATCH" and (props.material_type == "PLASTIC" or (props.material_type == "METAL" and props.metal_type == "NONE")):
            # === Scratch BSDF ===
            frame = nodes.new("NodeFrame")
            frame.label = "Scratch Imperfection"
            frame.name = "Scratch Imperfection"
            
            scratch_bsdf = nodes.new("ShaderNodeBsdfPrincipled")
            scratch_bsdf.location = (0, 0)
            scratch_bsdf.inputs["Base Color"].default_value = props.scratch_color
            scratch_bsdf.inputs["Metallic"].default_value = 1.0
            scratch_bsdf.parent = frame
            
            # === Bump Node ===
            bump = nodes.new("ShaderNodeBump")
            bump.location = (-300, 0)
            bump.inputs["Strength"].default_value = 0.2
            bump.invert = True
            links.new(bump.outputs["Normal"], scratch_bsdf.inputs["Normal"])
            bump.parent = frame
            
            # === ColorRamp for Scratch Mask ===
            ramp = nodes.new("ShaderNodeValToRGB")
            ramp.location = (-600, 0)
            ramp.color_ramp.elements[0].position = 0.665
            ramp.color_ramp.elements[1].position = 0.721
            links.new(ramp.outputs["Color"], bump.inputs["Height"])
            ramp.parent = frame
            
            # === Noise Texture ===
            noise = nodes.new("ShaderNodeTexNoise")
            noise.location = (-900, 0)
            noise.inputs["Scale"].default_value = 6
            noise.inputs["Detail"].default_value = 3.6
            links.new(noise.outputs["Fac"], ramp.inputs["Fac"])
            noise.parent = frame
            
            # === Voronoi Texture (Makoski) ===
            voronoi = nodes.new("ShaderNodeTexVoronoi")
            voronoi.location = (-1200, 0)
            voronoi.distance = 'MINKOWSKI'
            voronoi.feature = 'F1'
            voronoi.inputs["Scale"].default_value = 10
            voronoi.inputs["Exponent"].default_value = 0.5  # Makoski approximation
            links.new(voronoi.outputs["Distance"], noise.inputs["Vector"])
            voronoi.parent = frame
            
            # === Voronoi v2 + Mapping + TexCoord ===
            v2 = nodes.new("ShaderNodeTexVoronoi")
            v2.location = (-1800, -200)
            v2.feature = 'F1'
            v2.inputs["Scale"].default_value = 10
            v2.parent = frame
            
            mapping = nodes.new("ShaderNodeMapping")
            mapping.location = (-2100, -200)
            mapping.inputs["Location"].default_value = props.scratch_location

            mapping.inputs["Scale"].default_value = (
                props.scratch_size,
                props.scratch_size,
                props.scratch_size
            )
            mapping.parent = frame
            
            tex_coord = nodes.new("ShaderNodeTexCoord")
            tex_coord.location = (-2400, -200)
            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
            links.new(mapping.outputs["Vector"], v2.inputs["Vector"])
            tex_coord.parent = frame
            
            # === Mix Color between v2 and Object Coord ===
            mix_v = nodes.new("ShaderNodeMixRGB")
            mix_v.location = (-1500, -200)
            mix_v.blend_type = 'MIX'
            mix_v.inputs["Fac"].default_value = 0.5
            links.new(v2.outputs["Color"], mix_v.inputs["Color1"])
            links.new(tex_coord.outputs["Object"], mix_v.inputs["Color2"])
            mix_v.parent = frame
            
            # Connect mix_v to Voronoi vector
            links.new(mix_v.outputs["Color"], voronoi.inputs["Vector"])
            # === Additional Noise between mix_v and voronoi ===
            scratch_noise = nodes.new("ShaderNodeTexNoise")
            scratch_noise.location = (-1350, -100)
            scratch_noise.inputs["Scale"].default_value = 1.6
            scratch_noise.inputs["Detail"].default_value = 4.6
            scratch_noise.inputs["Roughness"].default_value = 0.380
            scratch_noise.parent = frame

            # Connect MixColor to Noise → then Noise to Voronoi
            links.new(mix_v.outputs["Color"], scratch_noise.inputs["Vector"])
            links.new(scratch_noise.outputs["Color"], voronoi.inputs["Vector"])

            # === Frame for Scratch Imperfection ===
            frame = nodes.new("NodeFrame")
            frame.label = "Scratch Imperfection"
            frame.name = "Scratch Imperfection"
            for n in [scratch_bsdf, bump, ramp, noise, voronoi, v2, mapping, tex_coord, mix_v]:
                n.parent = frame
            mix_shader = nodes.new("ShaderNodeMixShader")
            mix_shader.location = (300, 0)
            links.new(bsdf.outputs["BSDF"], mix_shader.inputs[1])         # User material
            links.new(scratch_bsdf.outputs["BSDF"], mix_shader.inputs[2]) # Scratch layer
            links.new(ramp.outputs["Color"], mix_shader.inputs["Fac"])    # Scratch mask
            links.new(mix_shader.outputs["Shader"], output.inputs["Surface"])
            links.new(bump.outputs["Normal"], bsdf.inputs["Normal"])            
            
        elif props.imperfections == "OXIDATION" and (props.material_type == "PLASTIC" or (props.material_type == "METAL" and props.metal_type == "NONE") or (props.material_type == "RUBBER" and props.rubber_type == "NONE") ):
            frame = nodes.new("NodeFrame")
            frame.label = "Oxidation Imperfection"
            frame.name = "Oxidation Imperfection"
            # ✅ Texture Coordinate Node
            tex_coord = nodes.new(type="ShaderNodeTexCoord")
            tex_coord.location = (-1400, 0)
            tex_coord.parent = frame
            
            # ✅ Mapping Node for customization
            mapping = nodes.new(type="ShaderNodeMapping")
            mapping.location = (-1200, 0)
            mapping.inputs["Location"].default_value = props.oxidation_mapping_location
            mapping.inputs["Scale"].default_value = (
            props.oxidation_mapping_scale,
            props.oxidation_mapping_scale,
            props.oxidation_mapping_scale
            )
            links.new(tex_coord.outputs["Object"], mapping.inputs["Vector"])
            mapping.parent = frame
            # ✅ Wave Texture for oxidation pattern
            wave = nodes.new(type="ShaderNodeTexWave")
            wave.location = (-1000, 0)
            wave.inputs["Scale"].default_value = 0.03
            wave.inputs["Distortion"].default_value = 70.0
            wave.inputs["Detail"].default_value = 15.0
            wave.inputs["Detail Scale"].default_value = 0.45
            wave.inputs["Detail Roughness"].default_value = 0.70
            wave.inputs["Phase Offset"].default_value = 7.67
            links.new(mapping.outputs["Vector"], wave.inputs["Vector"])
            wave.parent = frame
            
            # ✅ Noise Texture for fine oxidation details
            noise = nodes.new(type="ShaderNodeTexNoise")
            noise.location = (-800, 0)
            noise.inputs["Scale"].default_value = 4.82
            noise.inputs["Detail"].default_value = 6.5
            noise.inputs["Roughness"].default_value = 0.7
            links.new(wave.outputs["Fac"], noise.inputs["Vector"])
            noise.parent = frame
            
            # ✅ ColorRamp to refine oxidation mask
            ramp = nodes.new(type="ShaderNodeValToRGB")
            ramp.location = (-600, 0)
            ramp.color_ramp.elements[0].position = 0.527
            ramp.color_ramp.elements[1].position = 0.918
            links.new(noise.outputs["Color"], ramp.inputs["Fac"])
            ramp.parent = frame
            
            # ✅ Bump Node for oxidation surface texture
            bump = nodes.new(type="ShaderNodeBump")
            bump.location = (-400, 200)
            bump.inputs["Strength"].default_value = 0.25
            links.new(noise.outputs["Fac"], bump.inputs["Height"])
            bump.parent = frame
            
            # ✅ Oxidation BSDF shader
            oxid_bsdf = nodes.new(type="ShaderNodeBsdfPrincipled")
            oxid_bsdf.location = (0, 200)
            oxid_bsdf.inputs["Base Color"].default_value = (*props.oxidation_color, 1.0)
            oxid_bsdf.inputs["Roughness"].default_value = 0.8
            links.new(bump.outputs["Normal"], oxid_bsdf.inputs["Normal"])
            oxid_bsdf.parent = frame
            
            # ✅ Mix Shader to blend oxidation with base material
            mix_shader = nodes.new(type="ShaderNodeMixShader")
            mix_shader.location = (300, 0)
            links.new(bsdf.outputs["BSDF"], mix_shader.inputs[1])
            links.new(oxid_bsdf.outputs["BSDF"], mix_shader.inputs[2])
            links.new(ramp.outputs["Color"], mix_shader.inputs["Fac"])
            
            # ✅ Final Output
            links.new(mix_shader.outputs["Shader"], output.inputs["Surface"])
        
        
        # Only reassign if the current material is None or its name doesn't match
        if not obj.active_material or obj.active_material.name != mat_name:
            obj.data.materials.clear()
            obj.data.materials.append(mat)
            
        # If user clicked the "2" icon and renamed the material (e.g., .001), don't override it
        if obj.active_material and obj.active_material.name != mat_name:
            if not obj.active_material.name.endswith((".001", ".002", ".003")):
                obj.data.materials.clear()
                obj.data.materials.append(mat)
                obj["matricx_material_name"] = mat_name
        elif not obj.active_material:
            obj.data.materials.clear()
            obj.data.materials.append(mat)
            obj["matricx_material_name"] = mat_name


        
        context.view_layer.update()
        # ✅ Don't reassign if the user clicked "2" (i.e. made a local copy)
        current_mat = obj.active_material
        is_user_copy = current_mat and current_mat.name.startswith(mat_name) and current_mat.name != mat_name
        if not current_mat or (not is_user_copy and current_mat.name != mat_name):
            obj.data.materials.clear()
            obj.data.materials.append(mat)
            obj["matricx_material_name"] = mat_name

        self.report({'INFO'}, f"Material '{mat_name}' assigned to object.")
        return {'FINISHED'}
  
import bpy
from bpy.types import Panel

class PANEL_PT_MaterialGen(bpy.types.Panel):
    bl_idname = "PANEL_PT_matricx"
    bl_label = "MatricX"
    bl_category = "MatricX"
    bl_space_type = "VIEW_3D"
    bl_region_type = "UI"

    def draw(self, context):
        layout = self.layout
        layout.use_property_split = True
        layout.use_property_decorate = False

        props = context.scene.matgen_props
        obj = context.active_object
        pcoll = preview_collections.get("main")
        version_str = f"v{__version__[0]}.{__version__[1]}.{__version__[2]}"

        # ───────────── HELP / SYSTEM SECTION (Top) ─────────────
        layout.separator(factor=1.0)
        layout.prop(props, "show_help", toggle=True, icon='HELP')
        if props.show_help:
            box_sys = layout.box()
            box_sys.label(text=f"MatricX Version: {version_str}", icon='FILE_BLEND')
            box_sys.label(text="Stay Updated!", icon='INFO')
            row = box_sys.row(align=True)
            row.scale_y = 1.3
            row.operator("matricx.check_update", icon="URL", text="Check for Update")

        # ───────────── MATERIAL PREVIEW ─────────────
        if pcoll:
            # Map material_type to base preview name
            preview_map = {
                "METAL": "metal",
                "PLASTIC": "plastic",
                "WOOD": "wood",
                "RUBBER": "rubber",
                "EMISSION": "emission",
                "GLASS": "glass",
                "FLOOR": "floor",
                "SPECIAL_MAT": "special_mat"
            }
            base_preview = preview_map.get(props.material_type, "default")
            subtype = None

            if props.material_type == "METAL" and props.metal_type != "NONE":
                subtype = props.metal_type.lower()
            elif props.material_type == "WOOD" and props.wood_type != "NONE":
                subtype = props.wood_type.lower()
            elif props.material_type == "RUBBER" and props.rubber_type != "NONE":
                subtype = props.rubber_type.lower()
            elif props.material_type == "FLOOR" and props.floor_type != "NONE":
                subtype = props.floor_type.lower().replace("_", "")
            elif props.material_type == "SPECIAL_MAT" and props.special_mat_type != "NONE":
                subtype = props.special_mat_type.lower()
                if props.special_mat_type == "GOLD" and hasattr(props, "gold_type"):
                    gold_subtype = props.gold_type.lower().replace("type1", "procedural").replace("type2", "damaged_gold")
                    subtype = f"{subtype}_{gold_subtype}"
                elif props.special_mat_type == "CAMO" and hasattr(props, "camo_type"):
                    camo_subtype = props.camo_type.lower().replace("type", "type")
                    subtype = f"{subtype}_{camo_subtype}"
                elif props.special_mat_type in ["ALUMINUM", "CAR_PAINT", "COPPER", "SHADY"]:
                    subtype = props.special_mat_type.lower()

            # Construct preview image key
            img_key = f"{base_preview}_preview" if not subtype else f"{base_preview}_{subtype}_preview"
            if props.material_type == "SPECIAL_MAT" and props.special_mat_type == "GOLD" and img_key not in pcoll:
                if props.gold_type == "TYPE1":
                    img_key = "special_mat_procedural_preview"
                elif props.gold_type == "TYPE2":
                    img_key = "special_mat_damaged_gold_preview"
                
            elif props.material_type == "SPECIAL_MAT" and props.special_mat_type == "ALUMINUM" and img_key not in pcoll:
                img_key = "special_mat_aluminum_preview"  # Adjust if your image name differs

            if img_key in pcoll:
                layout.template_icon(icon_value=pcoll[img_key].icon_id, scale=5.0)
        layout.separator(factor=1.0)

        def draw_imperfection_settings(layout, props, imperfection_type, label_prefix=""):
            if imperfection_type == "DIRT":
                layout.prop(props, "dirt_color", text="Dirt Color")
                box_dirt = layout.box()
                box_dirt.label(text="Dirt Mapping", icon="WORLD")
                box_dirt.prop(props, "dirt_mapping_location", text="Location")
                box_dirt.prop(props, "dirt_mapping_scale", text="Scale")
                box_dirt.prop(props, "dirt_roughness", text="Roughness")
            elif imperfection_type == "DOWN_DIRT":
                layout.prop(props, "dirt_spread", text=label_prefix + "Spread")
                layout.prop(props, "dirt_opacity", text=label_prefix + "Opacity")
                layout.prop(props, "dirt_color_1", text=label_prefix + "Color 1")
                layout.prop(props, "dirt_color_2", text=label_prefix + "Color 2")
                layout.prop(props, "dirt_roughness", text=label_prefix + "Roughness")
                layout.prop(props, "dirt_metallic", text=label_prefix + "Metallic")
                layout.prop(props, "dirt_strength", text=label_prefix + "Bump Strength")
            elif imperfection_type == "RUST":
                layout.prop(props, "rust_spread", text=label_prefix + "Rust Spread")
                layout.prop(props, "rust_darkness", text=label_prefix + "Darkness")
            elif imperfection_type == "OXIDATION":
                layout.prop(props, "oxidation_color", text=label_prefix + "Oxid Color")
                layout.prop(props, "oxidation_mapping_scale", text=label_prefix + "Scale")
                layout.prop(props, "oxidation_mapping_location", text=label_prefix + "Location")
            elif imperfection_type == "EDGE_WEAR":
                layout.prop(props, "edge_wear_color", text=label_prefix + "Wear Color")
                layout.prop(props, "edge_wear_strength", text=label_prefix + "Wear Strength")
                layout.prop(props, "edge_wear_spread", text=label_prefix + "Wear Spread")
            elif imperfection_type == "SCRATCH":
                layout.prop(props, "scratch_color", text="Scratch Color")
                layout.prop(props, "scratch_size", text="Scratch Size")
                layout.prop(props, "scratch_location", text="")

        # ───────────── MATERIAL SETUP ─────────────
        layout.label(text="Material Setup", icon="MATERIAL")
        box_main = layout.box()
        box_main.prop(props, "material_type", text="Material")

        if props.material_type == "WOOD":
            box_main.prop(props, "wood_type", text="Wood Type")
       
        elif props.material_type == "RUBBER":
            box_main.prop(props, "rubber_type", text="Rubber Type")
            
        elif props.material_type == "SPECIAL_MAT":
            box_main.prop(props, "special_mat_type", text="Special Mat")
    
            if props.special_mat_type == "CAR_PAINT":
                box_main.prop(props, "car_paint_color", text="Paint Color")
        
            if props.special_mat_type == "GOLD":
                box_main.prop(props, "gold_type", text="Gold Type")

            if props.special_mat_type == 'CAMO':
                layout.label(text="Camo Material Settings")
                box_main.prop(props, "camo_type")
                box_main.prop(props, "pattern_randomness")
                box_main.prop(props, "camo_scale")
                
            if props.special_mat_type == "SHADY":
                box_main.label(text="Shady Ramp Colors", icon="COLOR")
                box_main.prop(props, "shady_color_1")
                box_main.prop(props, "shady_color_2")
                box_main.prop(props, "shady_color_3")
                box_main.prop(props, "shady_scale", text="Distortion Scale")

        elif props.material_type == "FLOOR":
            box_main.prop(props, "floor_type")

        if (props.material_type in ["PLASTIC", "EMISSION"]
            or (props.material_type == "RUBBER" and props.rubber_type == "NONE")
            or (props.material_type == "METAL" and props.metal_type == "NONE")):
            box_main.prop(props, "base_color", text="Base Color")

        if props.material_type == "METAL":
            box_main.prop(props, "metal_type", text="Metal Type")
            box_main.prop(props, "metal_crispness", text="Roughness")
        if props.material_type == 'METAL' and props.metal_type == 'BRUSHED':
            layout.prop(props, "brushed_type")
            if props.metal_type == "NONE":
                box_main.prop(props, "metal_roughness", text="Roughness")
        elif props.material_type == "EMISSION":
            box_main.prop(props, "emission_strength", text="Glow Strength")
        elif props.material_type == "PLASTIC":
            box_main.prop(props, "plastic_roughness", text="Roughness")
            box_main.prop(props, "plastic_glossiness", text="Gloss (Coat)")
        elif props.material_type == "GLASS":
            layout.label(text="Custom Glass Settings", icon='SHADING_RENDERED')
            box_main.prop(props, "glass_base_color", text="Base Color")
            box_main.prop(props, "glass_roughness", text="Roughness")
            box_main.prop(props, "smudge_strength", text="Smudge Imperfection")
            box_main.prop(props, "glass_texture_scale", text="Texture Scale")
            box_main.prop(props, "glass_crack_strength", text="Crack Amount")
            box_main.prop(props, "glass_crack_scale", text="Crack Scale")

        if props.material_type == "FLOOR":
            layout.label(text="Floor Settings", icon='GRID')
            box_floor = layout.box()
            if props.floor_type == "GRID":
                box_floor.prop(props, "grid_main_color")
                box_floor.prop(props, "grid_line_color")
                box_floor.prop(props, "grid_floor_scale")
            elif props.floor_type == "STATIC_BACKDROP":
                box_floor.prop(props, "backdrop_color", text="Backdrop Color")
                box_floor.prop(props, "backdrop_metallic", text="Metallic")
                box_floor.prop(props, "backdrop_roughness", text="Roughness")
            elif props.floor_type == "CONCRETE":
                box_floor.label(text="Concrete Options", icon="TEXTURE")
                box_floor.prop(props, "crack_detailing")
                box_floor.prop(props, "crack_strength")
                box_floor.prop(props, "concrete_crack_scale")

        if (props.material_type == "PLASTIC" or (props.material_type == "METAL" and props.metal_type == "NONE") or (props.material_type == "RUBBER" and props.rubber_type == "NONE") or (props.material_type == "SPECIAL_MAT" and props.special_mat_type == "CAMO")):
            layout.separator()
            layout.label(text="Imperfection Effects", icon="MOD_VERTEX_WEIGHT")
            box_imperf = layout.box()
            box_imperf.prop(props, "imperfections", text="Type")
            draw_imperfection_settings(box_imperf, props, props.imperfections)

        layout.separator()
        layout.label(text="Actions", icon="TOOL_SETTINGS")
        row = layout.row(align=True)
        row.operator("material.generate_smart", text="Generate", icon="CHECKMARK")
        row.operator("material.reset_material", text="Remove", icon="LOOP_BACK")
        # 🔧 Optional Fix Buttons
        if props.imperfections == "RUST":
            layout.operator("material.fix_rust_transform", text="Fix Rust Scale/Rotation", icon="TOOL_SETTINGS")
        elif props.imperfections == "EDGE_WEAR":
            layout.operator("material.fix_edgewear_transform", text="Fix Edge Wear Mapping", icon="MOD_UVPROJECT")


classes = (
    MaterialGeneratorProperties,
    OT_RemoveMaterial,
    OT_GenerateMaterial,
    OT_CheckForUpdate,
    PANEL_PT_MaterialGen,
)


def register():
    for cls in classes:
        bpy.utils.register_class(cls)
    bpy.utils.register_class(OT_FixRustTransform)
    bpy.utils.register_class(OT_FixEdgeWearTransform)
    bpy.types.Scene.matgen_props = bpy.props.PointerProperty(type=MaterialGeneratorProperties)
    
    # Initialize preview collection
    pcoll = bpy.utils.previews.new()
    preview_collections["main"] = pcoll
    
    # Load all preview images from material_preview folder
    addon_path = os.path.dirname(__file__)
    preview_dir = os.path.join(addon_path, "material_preview")
    if os.path.exists(preview_dir):
        for file in os.listdir(preview_dir):
            if file.endswith(".png"):
                img_name = os.path.splitext(file)[0]
                img_path = os.path.join(preview_dir, file)
                try:
                    pcoll.load(f"{img_name}_preview", img_path, 'IMAGE')
                except Exception as e:
                    print(f"Warning: Failed to load preview image {file}: {e}")
    else:
        print(f"Warning: material_preview directory not found at {preview_dir}")

def unregister():
    bpy.utils.unregister_class(OT_FixRustTransform)
    bpy.utils.unregister_class(OT_FixEdgeWearTransform)
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
    del bpy.types.Scene.matgen_props
    
    # Clean up preview collection
    for pcoll in preview_collections.values():
        bpy.utils.previews.remove(pcoll)
    preview_collections.clear()
    

if __name__ == "__main__":
    register()

