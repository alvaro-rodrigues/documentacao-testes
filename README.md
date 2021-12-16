**Nome:** Álvaro Rodrigues

**Matrícula:** 2017068718

**Projeto:** Django (https://github.com/django/django)

Django é framework web de alto nível baseado em Python que incentiva o desenvolvimento rápido e um design limpo e pragmático.

O framework possui o seu próprio mmódulo de testes que usa como base a bibilioteca unittest.

**Teste 1**
```python
# file options.py
def get_inline_instances(self, request, obj=None):
    inline_instances = []
    for inline_class in self.get_inlines(request, obj):
        inline = inline_class(self.model, self.admin_site)
        if request:
            if not (inline.has_view_or_change_permission(request, obj) or
                    inline.has_add_permission(request, obj) or
                    inline.has_delete_permission(request, obj)):
                continue
            if not inline.has_add_permission(request, obj):
                inline.max_num = 0
        inline_instances.append(inline)

    return inline_instances

# file tests.py
def test_inline_has_add_permission_uses_obj(self):
    class ConcertInline(TabularInline):
        model = Concert

        def has_add_permission(self, request, obj):
            return bool(obj)

    class BandAdmin(ModelAdmin):
        inlines = [ConcertInline]

    ma = BandAdmin(Band, AdminSite())
    request = MockRequest()
    request.user = self.MockAddUser()
    self.assertEqual(ma.get_inline_instances(request), [])
    band = Band(name='The Doors', bio='', sign_date=date(1965, 1, 1))
    inline_instances = ma.get_inline_instances(request, band)
    self.assertEqual(len(inline_instances), 1)
    self.assertIsInstance(inline_instances[0], ConcertInline)
```
- O teste em questão cria um modelo inline **ConcertInline** que extende a classe **TabularInline** ao invés de extender a classe **models.Model**. Em uma instância inline é possível setar o modelo a ser utilizado, **Concert** no caso, e na mesma classe já definir suas permissões. É necessário testar se um usuário com permissões de **add** consegue adicionar modelos na instância. Quando nenhum objeto é passado ao método **get_inline_instances** ele então deve retornar lista vazia. Já quando um objeto é passado então é retornado uma lista de tamanho 1, e como o modelo em questão é um **BandAdmin** que possui um inline definino como **ConcertInline**, então o último assert garante que a instância retornada na posição 0 da lista é um **ConcertInline**.

**Teste 2**
```python
# file sites.py
def register(self, model_or_iterable, admin_class=None, **options):
    # ...
    # If we got **options then dynamically construct a subclass of
    # admin_class with those **options.
    if options:

        options['__module__'] = __name__
        admin_class = type("%sAdmin" % model.__name__, (admin_class,), options)

    self._registry[model] = admin_class(model, self)

# file tests.py
def test_registration_with_star_star_options(self):
    self.site.register(Person, search_fields=['name'])
    self.assertEqual(self.site._registry[Person].search_fields, ['name'])
```
- Python e outras linguagens como Ruby e Scala tem suporte nativo a *keyword arguments*, que são argumentos extras para uma função. Em Pyhthon são precedidos por __**__ e comumente definidos como __**kwargs__. A função **register** possuir um *keyword argument* __**options__, e o teste em questão testa se esse argumento foi corretamente salvo após o regsitro.

**Teste 3**
```python
# file utils.py
def get_deleted_objects(objs, request, admin_site):
    # ...
    perms_needed = set()

    def format_callback(obj):
        # ...
        if has_admin:
            if not admin_site._registry[model].has_delete_permission(request, obj):
                perms_needed.add(opts.verbose_name)
    # ...

# file tests.py
def test_get_deleted_objects(self):
    mock_request = MockRequest()
    mock_request.user = User.objects.create_superuser(username='bob', email='bob@test.com', password='test')
    self.site.register(Band, ModelAdmin)
    ma = self.site._registry[Band]
    deletable_objects, model_count, perms_needed, protected = ma.get_deleted_objects([self.band], request)
    self.assertEqual(deletable_objects, ['Band: The Doors'])
    self.assertEqual(model_count, {'bands': 1})
    self.assertEqual(perms_needed, set())
    self.assertEqual(protected, [])
```
- O teste emula uma resquest usando o mock e em seguida seta um super usuário para a request. Em seguida é registrado um modelo **Band** que receberá as funcionalidades de um modelo **ModelAdmin**. É testado então se o método **get_deleted_objects**, atrelado ao objeto **Band**, retorna a saída correta em relação ao permissionamento do usuário. Como o usuário em questão é um super usuário então o objeto previamente definido de **Band** é deletável e nenhuma permissão a mais é necessária.
