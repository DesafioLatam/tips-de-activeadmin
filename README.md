# Tips de ActiveAdmin

Esta es una colección de tips para trabajar con [ActiveAdmin](https://activeadmin.info) en Ruby on Rails 
recogidos por el equipo de TI de [Desafío Latam](https://www.desafiolatam.com). 

## ¿Qué es Active Admin?
ActiveAdmin es un framework sobre Ruby on Rails que permite crear paneles de control de forma sencilla.
![](https://activeadmin.info/images/features.png)

## Crear un recurso

## Nombre y menú
Dentro del recurso del admin
1. Cambiar el nombre de un recurso

```
menu label: "Nombre"
```

2. Agregar un recurso a un Submenú

```
menu parent: :recurse_padre
```

3. Cambiar orden en el menú 
  - Los items se ordenan de mayor prioridad a menor.
  - Aplica también a los elementos del submenú.

```
menu priority: valor
```

4. Todo junto
```
menu :label => "Nombre", parent: :recurso_padre, priority: 40
```

## Scopes

Los scopes construyen automáticamente un filtro sobre un scope que ya existe en el modelo

1. Agrupar scopes
```
scope :nombre, group: :grupo
```

2. Cambiar el nombre de un scope
  Se puede cambiar el nombre de un scope utilizando un bloque
```
scope "Abierto", group: :admission_status do |recurso|
  recurso.un_scope_del_modelo
end
```

## Scoped collections

Se puede modificar la consulta utilizada por activeadmin, esto es útil para corregir problemas de n+1 query

```
controller do
  def scoped_collection
    end_of_association_chain.includes(:recurso_relacionado)
  end
end
```

## Guardar el current_user
Se puede sobreescribir el método create de inherited resources

```
controller do
  def create
    @enrollment = Enrollment.new(permitted_params[:enrollment])
    @enrollment.admin_user = current_admin_user
    @enrollment.save
  end
end
```

## Redirigir a una página distinta al crear o actualizar
```
controller do
  def create
    # Redirecciona al index del admind de program types
    create! { admin_program_types_url } 
  end 
  def update
    update! { admin_program_types_url }
  end 

end

```

## Filtros
Se puede remover un filtro específico de la barra lateral

```
remove_filter :filtro
```

## Index
Se puede agregar contenido html como links a una columna.
```
index do
  selectable_column
  column :id
  column "Modulos" do |g|
    raw "<a href='#{admin_generation_modules_path(q: {generation_id_eq: g.id})}'> #{g.generation_modules.length} </a>"
  end
```

## Formularios
1. Se puede agregar automáticamente un datepicker a un campo de fecha
```
form do |f|
  f.inputs do
    f.input :name
    f.input :admission_status
    f.input :start_date, as: :datepicker
  end
end
```

2. Se pueden customizar los selects
```
form do |f|
  f.inputs do
    f.input :name
    f.input :program_type, 
      collection: ProgramType.available.pluck(:name, :id), 
      include_blank: false
  end
end
```

3. Se puede mostrar un input solo si el formulario es para agregar un objeto (o solo para editar)
```
form do |f|
  if f.object.new_record? 
    f.input :name
  end
end
```



## Autenticar endpoints dentro de un controller que no sea Admin

```  
authenticate :admin_user, ->(admin_user) { !admin_user.nil? } do
  mount Blazer::Engine, at: "blazer"
  resources :enrollments
  post "enrollments/data-update", "enrollments#data_update"
end
```



