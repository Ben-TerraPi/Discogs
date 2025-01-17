# About it :  ![image](https://upload.wikimedia.org/wikipedia/commons/thumb/f/fe/Discogs-logo-billboard-1548-1092x722.jpg/320px-Discogs-logo-billboard-1548-1092x722.jpg)
Discogs is a database of information about audio recordings, including commercial releases, promotional releases, and bootleg or off-label releases.

Database contents are user-generated, and described in The New York Times as "Wikipedia-like".

While the site was originally created with the goal of becoming the largest online database of electronic music, it now includes releases in all genres and on all formats.

SITE: https://www.discogs.com/

API: https://api.discogs.com/

DOC:

https://python3-discogs-client.readthedocs.io/en/latest/index.html

https://www.discogs.com/developers/

# About me:

Je suis passionné de musique et collectionneur de vinyls, discogs est un site où je passe beaucoup de temps. Suite à une formation de Data Analyst, il était évident que pour mon premier projet personnel j'allais combiner ces deux centres d'intérêt.

Au départ ma volonté était de simplement lister ma collection personnel en utilisant python et l'API de discogs, il s'est vite avéré que je n'avais pas l'envie de m'arréter là.


# 1er dossier : [my_collection](https://github.com/Ben-TerraPi/Discogs/tree/main/my_collection) 

## scrap_discogs.py

J'ai commencé en utilisant **google collab** avec cette première ligne de code:

`! pip install python3-discogs-client`

C'était partie pour le début d'un projet qui m'a passionné.

### import

```
import discogs_client
import csv
import pandas as pd 
```

### Discogs Client & User token

Je me suis connecté à mon compte grâce à la génération d'un **token** développeur personnel qui m'a permis de naviguer dans l'API discogs.

```
d = discogs_client.Client("ExampleApplication/0.1", user_token= "secret")
me = d.identity()
```

### Mon compte

```
print(dir(me))
```

J'avais maintenant les attributs disponible pour mon compte client:

['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_known_invalid_keys', 'changes', 'client', 'collection_folders', 'collection_items', 'collection_value', 'data', 'delete', 'fetch', 'home_page', 'id', 'inventory', 'lists', 'location', 'name', 'num_collection', 'num_lists', 'num_wantlist', 'orders', 'previous_request', 'profile', 'rank', 'rating_avg', 'refresh', 'registered', 'releases_contributed', 'save', 'url', 'username', 'wantlist']

```
print(me.id)
print(me.location)
print(me.name)
print(me.url)
print(me.collection_folders)
```
A savoir:

* 2794711
* Rennes
* TerraPi
* https://www.discogs.com/user/Little.Red.Roquet
* [<CollectionFolder 0 'All'>, <CollectionFolder 1 'Uncategorized'>, ...

Avoir la liste de tous mes vinyles était aussi simple que cela:

```
data = []
for item in me.collection_folders[0].releases:
      data.append(item)
```
### Import de ma Collection

Il était temps de créer le tableau .csv avec les informations de mon choix:

| id  | title | artist | year | genre | style | master_id | release_country | labels | format | rating | have | want | url | image_url |
|-----|-------|--------|------|-------|-------|-----------|-----------------|--------|--------|--------|------|------|-----|-----------|

```
with open('collection.csv',
          'w',
          newline='',
          encoding='utf-8') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow([
        'id',
        'title',
        'artist',
        'year',
        'genre',
        'style',
        'master_id',
        'release_country',
        'labels',
        'format',
        'rating',
        'have',
        'want',
        'url',
        'image_url'
    ])

    for release in data:
        release_data = release.release

        genres = ", ".join(release_data.genres) if release_data.genres else "N/A"
        styles = ", ".join(release_data.styles) if release_data.styles else "N/A"
        master_id = release_data.master.id if release_data.master else "N/A"
        country = release_data.country if release_data.country else "N/A"
        labels = ", ".join([label.name for label in release_data.labels]) if release_data.labels else "N/A"
        formats = ", ".join([", ".join(fmt['descriptions']) for fmt in release_data.formats]) if release_data.formats else "N/A"
        community_rating = (
             release_data.community.rating.average if release_data.community and release_data.community.rating else "N/A"
        )
        community_have = release_data.community.have if release_data.community.have else "N/A"
        community_want = release_data.community.want if release_data.community.want else "N/A"
        image_url = release_data.images[0]["uri"] if release_data.images else "N/A"

        writer.writerow([
            release.id,
            release_data.title,
            release_data.artists[0].name if release_data.artists else "N/A",
            release_data.year,
            genres,
            styles,
            master_id,
            country,
            labels,
            formats,
            community_rating,
            community_have,
            community_want,
            release_data.url,
            image_url
        ])

print("Collection exportée dans 'collection.csv'.")
```
### Création du fichier final

Le scraping via l'API n'a pas nécissité un nettoyage conséquant. 

```
# load fichier .csv
df = pd.read_csv("/content/collection.csv")

# tri du tableau par nom artiste
df = df.sort_values("artist")

# rest de l'index du tableau
df = df.reset_index()

#suppression de l'ancienne colonne index
df = df.drop("index",axis=1)

#export du nouveau tableau .csv
df.to_csv("my_collection.csv")
```
Mon tableau était créé : [my_collection.csv](https://github.com/Ben-TerraPi/Discogs/blob/main/my_collection.csv)


## Stats_coll.py

Ayant découvert le commentaire **#%%** permettant des cellules de code Jupyter-like sur **VS code** j'arrète de travailler avec **google collab** et fais des tests dans cette nouvelle fenêtre interactive avec le fichier [Stats_coll.py](https://github.com/Ben-TerraPi/Discogs/blob/main/my_collection/Stats_coll.py) 


# 2ème dossier : [tests](https://github.com/Ben-TerraPi/Discogs/tree/main/tests)

S'ensuit une phase laborieuse de tests divers avec l'API.

Le contenu du site de discogs étant updater par sa communauté, son code est une accumulation de MAJ et à en croire les forums il est écrit par une multitude de codeur se succédant.

## fonctions de départ

De ce travail découlera une première fonction répondant à l'envie de chercher l'ensemble des albums d'un genre précis et d'une année précise:

```
def list_albums(genre, year):
    list = []
    results = d.search(genre=genre,year=year)
    for el in results:
        list.append(el)
    return pd.DataFrame(list)
```

En testant `list_albums("Hip Hop",1986)` le résultat est de **4156** lignes avec en ligne 0 <Master 42835 'Whodini - Funky Beat'>

Pourquoi un dataframe?
Un simple `return results` ne donne comme résultats qu'un message du genre: <discogs_client.models.MixedPaginatedList at 0x1bbcb513230>


## Fonction random_album

Maintenant que j'ai une liste exhaustive selon mes critères, et que j'ai compris que mes recherches sont regroupés dans un ensemble de pages, j'aimerai à l'image du bac à vinyles que l'on fouille tombé sur un album alétoirement:

```
import random    #rajouté à ma liste d'import au début du fichier

def random_album(genre, year):
    results = d.search(genre=genre,year=year)
    test = len(results)
    if test != 0:
        page_random = random.randint(1,results.pages)
        nb_results = len(results.page(page_random))
        k_random = random.randint(0,nb_results-1)
        return results.page(page_random)[k_random]
    else:
        return "todo"
```

Cette fois-ci je n'ai plus qu'un seul résultat comme par exemple: <Release 7261574 'Run DMC* - Walk This Way'>


## Fonction random_selecta

C'est bien d'avoir un album aléatoire, c'est encore mieux de pouvoir l'écouter!

```
def random_selecta(genre,style, year):
    results = d.search(genre=genre,style=style, year=year)
    test = len(results)
    if test != 0:

        #album aléatoire
        page_random = random.randint(1,results.pages)
        nb_results = len(results.page(page_random))
        k_random = random.randint(0,nb_results-1)
        album = results.page(page_random)[k_random]

        #info album
        title = album.title
        image = album.images[0]["uri"]
        link = album.url

        #youtube search
        str = title.lower()
        str2 = str.replace(" ","+")
        url = f'https://www.youtube.com/results?search_query={str2}'
        

        return title, image, url , link 
    else:
        return "todo"
```

Grâce à cette MAJ j'ai plus d'informations sur l'album sous forme de liens.

Exemple `random_selecta('hip hop', 'Boom Bap', 1986)` avec le résultat:

('Beastie Boys - (You Gotta) Fight For Your Right (To Party!)',
'https://i.discogs.com/9fJEL5bjdRIgKyxJjhK_R9TWVxgWYkBAebUmZA8IsrU/rs:fit/g:sm/q:90/h:600/w:584/czM6Ly9kaXNjb2dz/LWRhdGFiYXNlLWlt/YWdlcy9SLTY0MTA2/OC0xNjEwNjY3NDIx/LTEyMDEuanBlZw.jpeg',
 
'https://www.youtube.com/results?search_query=beastie+boys+-+(you+gotta)+fight+for+your+right+(to+party!)',
 
'https://www.discogs.com/master/1007458-Beastie-Boys-You-Gotta-Fight-For-Your-Right-To-Party')


# Création d'un STREAMLIT 

Le but est d'avoir une interface utilisateur simple où l'on sélectionne un **genre**, un **style** et une **année** avant de générer la requête.
 
Au cours de ma formation de Data Analyst j'ai realisé un projet sur streamlit et pour ce que je souhaite faire cette application est suffisante.

Je reste donc sur VS code et créé un **repository github** en lien avec mon application Streamlit.



## Set-up [.streamlit](https://github.com/Ben-TerraPi/Discogs/tree/main/.streamlit)

Rien de compliqué ici, le fichier config.toml pour la charte graphique mais surtout un fichier **secrets.toml** caché par **.gitignore** pour garder privé ma clé API discogs et ultérieurement une autre clé YouTube.

Et [requirements.txt](https://github.com/Ben-TerraPi/Discogs/blob/main/requirements.txt) à la racine du projet.

# 3ème dossier : [random_selecta](https://github.com/Ben-TerraPi/Discogs/tree/main/random_selecta)

Dans ce dossier on retrouve:

* [utils.py](https://github.com/Ben-TerraPi/Discogs/blob/main/random_selecta/utils.py) qui regroupe toutes mes fonctions.

* [Random_title.py](https://github.com/Ben-TerraPi/Discogs/blob/main/random_selecta/Random_title.py) qui sert au lancement de l'application Streamlit.

* [list_styles.py](https://github.com/Ben-TerraPi/Discogs/blob/main/random_selecta/list_styles.py) dans lequelle j'ai créé un dictionnaire pour chaques styles musicaux présent dans chaques genres musicaux référencés par Discogs, cela sera intégré pour les selectbox sur streamlit.

```
genres_styles = {"Blues" : blues_style,
  "Brass & Military " : military_style,
  "Children's" : children_style,
  "Classical" : classical_style, 
  "Electronic" : electro_style ,
  "Folk, World, & Country" : world_style, 
  "Funk / Soul" : funk_soul_style,
  "Hip Hop" : hip_hop_style, 
  "Jazz" : jazz_style, 
  "Latin" : latin_style,
  "Non-Music" : non_music_style,
  "Pop" : pop_list,
  "Reggae" : reggae_style,
  "Rock" : rock_style, 
  "Stage & Screen" : stage_style}

#Création tableau
df = pd.DataFrame.from_dict(genres_styles, orient='index').transpose()

df.to_csv("tableau_genre.csv")
```
Est créé le [tableau_genre.csv](https://github.com/Ben-TerraPi/Discogs/blob/main/tableau_genre.csv)

## Fonction final

```
def random_youtube(genre, style, year):

    results = d.search(genre=genre,style=style, year=year)
    test = len(results)

    #Variable par défaut
    title = None
    image = None
    url = None
    link = None
    discogs_videos = None
    youtube_results = None

    if test != 0:

        #album aléatoire
        page_random = random.randint(1,results.pages)
        nb_results = len(results.page(page_random))
        k_random = random.randint(0,nb_results-1)
        album = results.page(page_random)[k_random]

        #info album
        title = album.title
        if hasattr(album, 'images') and album.images:
            image = album.images[0]["uri"]
        link = album.url


        #youtube search
        str = title.lower()
        str2 = str.replace(" ","+")
        str3 = str2.replace("&","and")
        url = f'https://www.youtube.com/results?search_query={str3}'

        #Vidéo Discogs
        if hasattr(album, 'videos') and album.videos:
            discogs_videos = album.videos[0].url

        else :
            # Recherche sur YouTube via l'API YouTube
            youtube = build('youtube', 'v3', developerKey=api_key)
            request = youtube.search().list(
                part="snippet",
                q=str,
                type="video",
                maxResults=1  # Limiter le nombre de résultats
            )
            try:
                response = request.execute()
            except:
                response = None

            # Extraire les résultats de la recherche YouTube
            youtube_results = []
            
            if response:

                for item in response['items']:
                    video_id = item['id']['videoId']
                    video_title = item['snippet']['title']
                    video_url = f"https://www.youtube.com/watch?v={video_id}"
                    youtube_results.append({'title': video_title, 'url': video_url})

        return title, image, url , link , discogs_videos , youtube_results
```

# Conclusion

Arriver à ce stade je vous laisse analyser la struture du fichier streamlit [Random_title.py](https://github.com/Ben-TerraPi/Discogs/blob/main/random_selecta/Random_title.py)  et voir le résultat.

https://discogs-random-selecta.streamlit.app/















