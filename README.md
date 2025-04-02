---
author: Xevi Baró, Kaisar Kushibar, Eloi Puertas
date: Abril 2025
title: Pràctica 2 - FastAPI and Vue application (Software distribuït)
---

# Enunciat Pràctica 2

# Introducció

Seguint la pràctica 1, en aquesta pràctica implementarem el joc d'enfonsar la flota, però ara utilitzarem protocols estàndard web i una base de dades.
Treballarem amb dos mòduls completament diferenciats:

- **Backend**: Implementarà una REST-API per accedir i gestionar de forma segura les dades de l'aplicació. S'encarregarà de l'autenticació dels usuaris i de l'accés a la base de dades.
- **Frontend**: Consisteix en una aplicació web que s'executarà localment en el navegador de l'usuari, permetent la interacció de l'usuari amb l'aplicació.

## Important

Aquest exercici guiat suposa que ja teniu alguna versió de Python de versió mínima `3.10`
instal·lada i esteu familiaritzats amb la instal·lació de paquets mitjançant `pip`.

Tingueu en compte que `la comanda pip` mostrada en aquest tutorial correspondrà a `pip3` segons quantes versions diferents de python tingueu
instal·lat al vostre ordinador.

Es recomana altament l'ús de la IDE [PyCharm Professional](https://www.jetbrains.com/pycharm/) per al desenvolupament d'aquesta pràctica. Si no el teniu instal·lat, existeix una versiñó per a estudiants que podeu obtenir si us enregistreu amb el correu de la UB.

## Enviament dels Exercicis

Es treballarà de la mateixa manera que a la Pràctica 1, utilitzant un repositori de `GitHub Classroom` com a base. El repositori ha de contenir:

- El codi que implementa totes les funcionalitats requerides.
- `Pull Requests` setmanals amb:
  - la descripció de les tasques complertes durant la setmana i les que han quedat pendents.
  - organització dels membres de l'equip per a la realització de les tasques
  - problemes sorgits durant la setmana
  - resum de proves realitzades
- Durant la realització de la pràctica haureu d'anar documentant el vostre progrés en la memòria que trobareu en el directori `docs` del template de la pràctica.

**NOTA:** Recordeu que els `Pull Requests` cal que els iniciï un membre de l'equip i els accepti l'altre, donant així conformitat al seu contingut per part dels dos membres de l'equip.

## Avaluació pràctica 2.

- 20% pull requests setmanals
- 80% Lliurament final:
  - 20% Sessió de Test,
  - 20% Proves unitàries
  - 50% codi final,
  - 10% Memòria progrés.

## Objectius de la Pràctica

#### Back-End:

- Crear aplicació CRUD amb autentificació per a gestionar (CRUD: Create, Read, Update i Delete) de la part de model d'aplicació
- Gestionar la lògica del joc

#### Front-End:

- Interacció amb l'usuari, tant pel que fa a la visualització de dades com a la recollida d'accions.
- Comunicació segura amb el back-end

# Calendari

| Sessió | Data              | Tema                                  | Guia                             | Enllaços                                                                  |
| ------ | ----------------- | ------------------------------------- | -------------------------------- |---------------------------------------------------------------------------|
| 1      | 02/04/25          | Desplegament local backend i frontend | [Sessió 1](Sessions/Sessio_1.md) | [Intro a Vue](Guies/inici_Vue.md) [Intro a Django](Guies/inici_DJango.md) |
| 2      | 23/04/25-30/04/25 | Backend i base de dades               |                                  |                                                                           |
| 3      | 07/05/25          | **Qüestionari 2** i Frontend          |                                  |                                                                           |
| 4      | 14/05/25          | Autenticació i autorització           |                                  |                                                                           |
| 5      | 21/05/25          | Taula de lideratge                    |                                  |                                                                           |
| 6      | 28/05/25          | Sessió de Testing                     |                                  |                                                                           |
