# Inicialització del backend amb DJango

## Crear entorn virtual de Python
La gestió de les dependències la farem utilitzant l'eina [Poetry](https://python-poetry.org/). Aquesta eina
permet definir les llibreries necessàries i crear un entorn virtual de Python amb aquestes llibreries i totes les
seves dependències. D'aquesta forma, evitem que hi hagi conflictes amb altres projectes de Python.

Per inicialitzar el projecte s'utilitza la comanda:

```bash
poetry new backend
```

que genera l'estructura bàsica del projecte **backend**. El fiter **pyproject.toml** conté tota la informació sobre el
projecte, inicialment informació molt bàsica com el nom, versió i descripció del nostre projecte. En el nostre cas no
estem creant un nou paquet de Python, sinó una aplicació, per tant afegirem la següent línia en el fitxer de configuració:

```toml
[tool.poetry]
package-mode = false
```
També podem eliminar la carpeta **backend** que conté el paquet buit creat incialment per Poetry.

A partir d'aquí, cal entrar dins el nou directori (on hi ha el fitxer pyproject.toml) i afegir els paquets necessàris. 
Afegirem els paquests principals amb la comanda:

```bash
poetry add django djangorestframework drf-spectacular
```

De la mateixa manera, també podem afegir paquets que només necessitem per a la fase de desenvolupament, com els que ens
serveixen per passar les proves al codi o comprovar la seva correctesa. Podem afegir també aquestes dependències amb la
comanda:

```bash
poetry add pytest pylint allure-pytest --group dev
```

Per tal de treballar en l'entorn virtual gestionat per Poetry, ho podem fer de dues maneres:

1. Activar l'entorn amb ```poetry shell```, que farà que s'utilitzi l'entorn virtual de python en tot el que fem a
partir d'aquest moment, fins que executem ```deactivate```.
2. Utilitzar explicitament l'entorn, afegint ```poetry run``` abans de cada comanda que es vulgui executar dins de
l'entorn virtual. Per exemple, ```poetry run python --version``` executaria la comanda ```python --version``` dins de 
l'entorn virtual.

Recorda utilitzar un d'aquests mètodes en la resta de la guia.

## Inicialitzar el projecte DJango
[DJango](https://www.djangoproject.com/) és un entorn per al desenvolupament d'aplicacions en python. Per crear el projecte inicial, caldrà executar la següent comanda (cal haver eliminat la carpeta **backend** en el pas 
anterior, o tindrem conflictes):

```bash
django-admin startproject backend
```

Aquesta comanda ens haurà tornat a crear una carpeta backend, amb un fitxer **manage.py** el seu interior (entre altres coses).
El fitxer **manage.py** és el que ens permet interactuar amb DJango. A continuació el farem servir per tal de preparar
la base de dades i arrancar el projecte.

### Configuració

Tota la configuració de DJango es fa a través d'un fitxer de configuració. Aquest fitxer el podeu trobar a
```/backend/backend/backend/settings.py```. A mesura que anem afegint components al nostre projecte, caldrà
afegir la configuració pertinent.

Podem afegir funcionalitats al DJango via **aplicacions**, ja sigui les que fem nosaltres com
les que ja existeixen. Per començar, indicarem a DJango que volem que utilitzi dues aplicacions
que hem instal·lat prèviament amb el django. Per fer-ho, modificarem la llista d'aplicacions instal·lades
**INSTALLED_APPS** dins el fitxer de configuració, afegint ```rest_framework``` i ```drf_spectacular``` al final:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'drf_spectacular',
]
```

### Creació de la base de dades
DJango ens proporciona un sistema de gestió de migracions de la base de dades molt potent, que permet gestionar directament
la creació i actualització de la base de dades per adaptar-la a la nostra aplicació, però també a totes les aplicacions
existents que li afegim. Veurem el tema de migracions a la pràctica, de moment simplement aplicarem les que ja existeixen 
amb la comanda:

```bash
python manage.py migrate
```

### Iniciar el servidor

Un cop tenim la base de dades creada, ja podem comprovar que el nostre servidor funciona. Per arrancar el servidor,
cal executar la comanda:
```bash
python manage.py runserver
```

Si tot ha anat bé, haurieu de poder accedir amb un navegador a l'adreça 
[http://127.0.0.1:8000/](http://127.0.0.1:8000/), i veure la pàgina inicial de DJango.


## Afegir una nova aplicació
Un cop tenim la nostra aplicació bàsica funcionant, ara 