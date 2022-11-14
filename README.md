# Tips de ActiveAdmin

## Nombre y menú
- Cambiar el nombre de un recurso:
  ```ruby
  menu label: "Nombre"
  ```

- Agregar un recurso a un Submenú:
  ```ruby
  menu parent: :recurse_padre
  ```

- Cambiar orden en el menú: 
  - Los items se ordenan de mayor prioridad a menor.
  - Aplica también a los elementos del submenú.
  ```ruby
  menu priority: valor
  ```

- Cambiar todo a la vez
  ```ruby
  menu :label => "Nombre", parent: :recurso_padre, priority: 40
  ```
## Scopes

Los scopes construyen automáticamente un filtro sobre un scope de que ya existe en un modelo

- Agrupar scopes:
  - Los botones agrupados aparecen juntos en la interfaz
  ```ruby
  scope :nombre, group: :grupo
  ```

- Cambiar el nombre de un scope:
  - Se puede cambiar el nombre de un scope utilizando un bloque

  ```ruby
  scope "Abierto", group: :admission_status do |recurso|
    recurso.un_scope_del_modelo
  end
  ```

- Crear un scope activado por defecto
```ruby
  scope :nombre, default: true
```

- Crear un botón de un scope para seleccionar todos los recursos
  ```ruby
  scope :all
  ```

## Scoped collections
Se puede modificar la consulta utilizada por activeadmin, esto es útil para corregir problemas de N + 1 query

```ruby
controller do
  def scoped_collection
    end_of_association_chain.includes(:recurso_relacionado)
  end
end
```

## Guardar el current_user
Se puede sobreescribir el método create de inherited resources

```ruby
controller do
  def create
    @enrollment = Enrollment.new(permitted_params[:enrollment])
    @enrollment.admin_user = current_admin_user
    @enrollment.save
  end
end
```

## Redirigir a una página distinta al crear o actualizar
```ruby
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
- Se puede remover un filtro específico de la barra lateral
  ```ruby
  remove_filter :filtro
  ```

- Se puede agregar un filtro con un método custom de ransack
  ```ruby
    # En el admin
    filter :search_in_last_admission_state, 
      as: :searchable_select, 
      collection:  proc{ AdmissionState.all.collect { |as| [ as.name, as.id]} }

    # En el modelo del admin respctivo
    def self.ransackable_scopes(auth_object = nil)
      [:nombre_del_metodo_para_filtrar]
    end

    scope :nombre_del_metodo_para_filtrar, -> (criterio_adicional = nil) {
      .where("criterio_adicional IN (?)") # Útil para filtros complejos 
    }
  ```

## Index
Se puede agregar contenido HTML como links a una columna.
```ruby
index do
  selectable_column
  column :id
  column "Modulos" do |g|
    raw "<a href='#{admin_generation_modules_path(q: {generation_id_eq: g.id})}'> #{g.generation_modules.length} </a>"
  end
```

## Formularios
- Se puede agregar automáticamente un datepicker a un campo de fecha
  ```ruby
  form do |f|
    f.inputs do
      f.input :name
      f.input :admission_status
      f.input :start_date, as: :datepicker
    end
  end
  ```

- Se pueden customizar los selects
  ```ruby
  form do |f|
    f.inputs do
      f.input :name
      f.input :program_type, 
        collection: ProgramType.available.pluck(:name, :id), 
        include_blank: false
    end
  end
  ```

- Se puede mostrar un input solo si el formulario es para agregar un objeto (o solo para editar)
  ```ruby
  form do |f|
    if f.object.new_record? 
      f.input :name
    end
  end
  ```

## Autenticar endpoints dentro de un controller que no sea Admin
```ruby
authenticate :admin_user, ->(admin_user) { !admin_user.nil? } do
  mount Blazer::Engine, at: "blazer"
  resources :enrollments
  post "enrollments/data-update", "enrollments#data_update"
end
```
