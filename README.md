# Tips de ActiveAdmin
> Esta es una colección de tips para trabajar con [ActiveAdmin](https://activeadmin.info) en Ruby on Rails 
recogidos por el equipo de TI de [Desafío Latam](https://www.desafiolatam.com). 

## ¿Qué es Active Admin?
> ActiveAdmin es un framework sobre Ruby on Rails que permite crear paneles de control de forma sencilla.

![](https://activeadmin.info/images/features.png)

## Crear un recurso nuevo
rails generate active_admin:resource nombre_del_modelo

## Crear una página nueva
1. Crear un archivo dentro de admin, ejemplo: enrollment_stats.rb
2. Registrar la página 
```ruby
ActiveAdmin.register_page 'EnrollmentStats' do
  content title: "título" do
    # Aquí puedes agregar componentes Arbre y/o renderear una vista parcial
  end
end
```

## Nombre y menú
Dentro del admin del recurso:

- Cambiar el nombre de un recurso:
  ```ruby
  menu label: "Nombre"
  ```

- Agregar un recurso a un Submenú:
  ```ruby
  menu parent: :recurse_padre
  ```

- Cambiar orden en el menú 
  - Los items se ordenan de menor prioridad a mayor. 
  - De la misma forma se pueden ordenar los elementos del submenú.

  ```ruby
  menu priority: 10
  ```

- Todo junto
  ```ruby
  menu :label => "Nombre", parent: :recurso_padre, priority: 40
  ```
## Scopes
Los scopes construyen automáticamente un filtro sobre un scope que ya existe en el modelo.

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

## Scopes
Los scopes construyen automáticamente un filtro sobre un scope que ya existe en el modelo.

1. Agrupar scopes

```ruby
scope :nombre, group: :grupo
```

2. Cambiar el nombre de un scope
Se puede cambiar el nombre de un scope utilizando un bloque

```ruby
scope "Abierto", group: :admission_status do |recurso|
  recurso.un_scope_del_modelo
end
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

## Guardar el `current_user`
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
    raw "<a href='#{admin_generation_modules_path(q: {generation_id_eq: g.id})}'> 
      #{g.generation_modules.length} 
    </a>"
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

- Se pueden introducir datos de un modelo relacionado directamente en el formulario. Supongamos que tenemos el modelo de estudiante y el modelo de notas. Un estudiante tiene muchas notas.

  1. Agregamos el input en el admin de estudiante
  ```ruby
    form do |f|
      f.input :name
      f.has_many :others do |other|
        other.input :score
      end
  end
  ```

  2. Indicamos en el modelo de estudiante que puede recibir información del objeto relacionado
  ```ruby
  accepts_nested_attributes_for :scores, allow_destroy: true
  ```

  3. Agregamos en el admin de estudiante los strong params
  ```ruby
    permit_params :name,
      :scores_attributes[:score]
  ```


## Crear diversos paneles
Se pueden dividir el index, show o incluso el dashboard en múltiples columnas
```ruby
  show do
    columns do
      column do 
        attributes_table do
          ...
        end
      end
      column do
        h3 "Otra información"
      end
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

## Dar acceso a una página genérica utilizando cancancan
```ruby
if user.rol == "rol_con_acceso"
  can :read, 
    ActiveAdmin::Page, 
    name: "Dashboard", # Dar acceso al dashboard
    namespace_name: "admin"
  can :read, 
    ActiveAdmin::Page, 
    name: "OtraPaginaRegistrada", # Dar acceso a otra página
    namespace_name: "admin"
```
