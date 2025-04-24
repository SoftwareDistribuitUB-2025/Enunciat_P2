# 🧠 Django Game API - Distributed Development Guide

Welcome to the **Game API Project**! This guide is designed for undergraduate students participating in a **distributed software development activity**. You'll learn how to collaboratively build a RESTful API using **Django** and **Django REST Framework (DRF)**.

---

## 🌐 What Is an API?

An **API (Application Programming Interface)** allows different software systems to communicate. In this project:

- The **backend** exposes resources like `Player` and `Game`.
- The **frontend** or another service consumes those via HTTP (GET, POST, PUT, DELETE).

---

## 🛠️ Project Structure

| Component        | Purpose                                       |
| ---------------- | --------------------------------------------- |
| `models.py`      | Defines database schema (Player, Game)        |
| `serializers.py` | Translates models <-> JSON data               |
| `views.py`       | Handles API logic and permissions             |
| `permissions.py` | Custom rules for who can access what          |
| `urls.py`        | Maps API routes to views                      |
| `signals.py`     | Auto-creates Player when a new User registers |

---

## 🎓 Key Concepts

### 1. Models

Define your database structure:

```python
class Player(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    nickname = models.CharField(max_length=50)
```

### 2. Serializers

Convert models to/from JSON for APIs:

```python
class PlayerSerializer(serializers.ModelSerializer):
    class Meta:
        model = Player
        fields = '__all__'
```

### 3. ViewSets

Handle CRUD operations:

```python
class GameViewSet(viewsets.ModelViewSet):
    queryset = Game.objects.all()
    serializer_class = GameSerializer
```

### 4. Permissions

Restrict access (e.g., only game owners can edit):

```python
class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.owner.user == request.user
```

### 5. Custom Actions

Add domain-specific features:

```python
@action(detail=True, methods=['post'])
def join(self, request, pk=None):
    ...
```

### 6. Signals

Create Player when a new User is registered:

```python
@receiver(post_save, sender=User)
def create_player(sender, instance, created, **kwargs):
    if created:
        Player.objects.create(user=instance, nickname=instance.username)
```

---

## 🌍 Distributed Collaboration Tips

- ✍️ **Use Git**: Commit small, frequent changes.
- ✉️ **Document decisions**: Use issues or markdown files.
- 💬 **Communicate actively**: Discuss logic or conflicts.
- 👥 **Respect roles**: Backend, frontend, QA, etc.

---

## 🌌 Workflow Summary

```text
Frontend (or user)
   ⬇
HTTP Request (e.g., GET /games/)
   ⬇
Router (urls.py)
   ⬇
ViewSet (views.py)
   ⬇
Serializer (serializers.py)
   ⬇
Model (models.py)
   ⬆
Database
   ⬆
HTTP Response (JSON)
```

---

## ✅ Key Endpoints

| Method | URL                      | Description              |
| ------ | ------------------------ | ------------------------ |
| GET    | `/api/players/`          | List players             |
| POST   | `/api/games/`            | Create game (auto owner) |
| POST   | `/api/games/{id}/join/`  | Join a game              |
| POST   | `/api/games/{id}/start/` | Start a game             |

---

## 🎓 Learning Resources

- Django: [https://docs.djangoproject.com/](https://docs.djangoproject.com/)
- DRF: [https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
- Postman: [https://www.postman.com/](https://www.postman.com/)
- GitHub: [https://docs.github.com/en](https://docs.github.com/en)

---

