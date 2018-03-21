# Récupérer des éléments depuis la base de données

On utilise pour cela *Doctrine* : 
```php
$user = this->getDoctrine()
->getRepository(Users::class)->findAll();

return this->render('admin/users.html.twig', ['user' => $user])
```

Ensuite au niveau de la vue il ne nous reste plus qu'a lister les différents élements récupérés :
```twig
{% for u in user %}
  {{ u.username }}
  {{ u.email }}
  {{ u.roles }}
{% endfor %}
```