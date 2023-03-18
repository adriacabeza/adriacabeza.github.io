One of the most feared parts about getting a job as a Software Engineer is the technical interview. Some of the most wanted high-tech companies like Google, Microsoft or Amazon are known for having very complex and almost impossible algorithmic interviews.

This post is not going to be a rant about those practices: I get them, asking about data structures and algorithms can help companies weed out the bad programmers who only can copy+paste good answers at StackOverflow wihout understanding them. But still, imho this interviewing method can be improved a lot. Asking how to reverse a tree is not going to tell you who is the best frontend developer, that's just nonsense. 

This post is about another type of interviewing process, more concretely, Shopify's Backend Developer Internship 2021 interviewing process: 
<hr>
<i>TASK: Build an image repository.</i>

<i>You can tackle this challenge using any technology you want. This is an open-ended task.</i>
<i>Please provide brief instructions on how to use your application. We are a test driven environment, so ensure your challenge includes tests. Provide brief instructions on how to use your application, so that someone could use your application and understand how it works.</i>
<hr>

UUUhhh... that's quite interesting! I would love to see a world where all the software interviewing processes are similar to this one. This allows the applicant to be creative, to show his stack skillset, to design a pretty complete project... 

Moving forward, I have been looking for a cool project to work with for a long time (I could finish 1 of the 10 unfinished projects that I have but that does not goes with my philosophy XD) so inevitably I started to think about all the cool things that could be implemented (except the required MVP functionalities like adding photos, deleting, etc):

- **Similarity search**: by using a nice hash function (phash or dhash) or some kind of feature extraction and a similarity engine or a cool data structure like VP trees. 
- **Concept search**: if I could extract information of the image (with some image classification model like Resnet50) I'd be able add a search function similar to the one from Google Photos, i.e. if you are looking for your cat photos, just type in cat and those photos will magically appear.
- **Cloud ready**: at first glance I'd consider a local app to manage your files, but what if I wanted it to be in the cloud? This will make the things more difficult. Users, authentification, cloud storage...
- **Async code**: Most of the image processing functionalities should be async... should I go for an event architecture to separe the API and these functionalitites? 

As you can see, the possibilities are huge. So after thinking about it for a moment. I decided what it was obvious from the initial moment:

Yeah, I am doing it. 

## Planning the implementation

Before jumping into the implementation, I spent a week of research to come up with the best possible software design and the tech stack that mosts fits the needs of the project. 

Moreover, I have tried to choose a good balance of well-known and never used technologies. For example, I already know how to work with Docker-compose, Tensorflow or FastAPI, but I have never used Celery, RabbitMQ or Jinja. 

Here you can see the main list:
- Docker-Compose
- Celery
- RabbitMQ
- Tensorflow
- NMSLib
- FastAPI
- Jinja
- PostgreSQL
- Uvicorn


In terms of Software Design, even though I want to start simple, I will try to come up with a codebase that makes it easy to scale and implement new functionalities. For now, I will start implementing it as it was in the localsystem by sharing the same data volume between docker instances. Whenever this is working, the next step will be to move it to the cloud to a service like S3 or Google Cloud Storage. Moreover, when that occurs, we will need some kind of user logic (register, login, tokens, etc...)

The plan is to use an events architecture (using a Task Queue) that has a completely separated instances of an API and a "Brain". This will remove the problem of having to wait to for long tasks like the machine learning based ones. 

As a summary, we would have something like this:

<div align="center">
  <img src="http://lordsantanna.com/img/image-repository.png">
</div>

# Too many features

As a feraceous hacker that I am (Long Live to HackUPC!!) I tend to come up with MVPs pretty fast. The downside is that sometimes the best practices are forgotten in the process (aka tests, not leaving unfinished features, etc). However, during this initial development, my main problem were not this bad habits I gained along every hackathon, my main problem was the amount of features I wanted to tackle. I had so many ideas around my head... and I wanted to implement them all so... I started doing so, which obviously is a very bad idea. It normally ends with an enormous set of unfinished features and a never-ending project. 

Just to show you how big of a problem this was, I even started to code it to admit different users (not deploying it in the cloud was not even a thought):

```python
# Database I would have used to relate users and images
users = Table(
    "users",
    metadata,
    Column("email", String(70), primary_key=True, unique=True),
    Column("hashed_password", String(100), nullable=False),
    Column("name", String(50)),
    Column("images", Integer, ForeignKey("images.id"), nullable=False)
)

# Schema used in the API to deal with Users
class User(BaseModel):
    """
    This schema contains the information of a client.
    """
    name: str
    email: str
    password: str
    images: int
    full_name: Optional[str] = None

    class Config:
        schema_extra = {
            "example": {
                "name": "Shoto",
                "password": "itachi",
                "email": "mail1@test.com",
                "full_name": "Shoto Todoroki",
                "images": 1,
            }
        }
```
I even had implemented OAuth2 and JWT to deal with them!

```python
# Schemas that deal with access tokens
class Token(BaseModel):
    """"
    This is a token from JWT
    """
    access_token: str
    token_type: str


class TokenData(BaseModel):
    """
    This schema is used only to get the username from the token
    """
    username: Optional[str] = None
```


All this madness continued for two days until I realized (while listening to this wonder: https://open.spotify.com/track/29T3H5d9TgTlYcKiuNxdRT?si=361b9f82bc574841) that this was not what I wanted. I deleted everything that was annoying me, I took a paper and started to write a plan (in a very specific way): which are the most important features?, which order should I follow when implementing those? and which information I need to save.

These will be the main features that I will try to implement (in that order):
- Add images (it will be a POST method in the API)
- Delete images (as a DELETE method in the API)
- Display them in a nice gallery (using Jinja templates)
- Search them by concept (with a constrained set of objects)
- Search them by similarity (it will be a GET method in the API)
  - Implement the feature extraction/hashing and the creation of the similarity index

In terms of database, I will have one with simple image data like image name, image path or thumbnail path, and another one for the categories used in the image search capabilities. 

For the second one, my first thought was to save for every image its predicted list of categories:
```
image_1: sky, dog
image_2: dog, table
image_3: sea, sky
```

but if a user searches for a word, we would have to traverse all the pictures to retrieve the desired images which is a pretty expensive call. Instead, we will add another table that serves us as an inverted index. So, for every category we store a list of images with positives for that category:
```
sky: image_1, image_3
dog: image_1, image_2
table: image_2
sea: image_3
```


# Designing how to search

### Text search

Let's be real here: nobody searches images by their filename. If you want to look for a photo from you last trip to Zanzibar, you will not type the filename set by your camera. Probably you'd scroll down to the date when the photos were took and then look at individual photos or thumbnails to try to identify it. 

Wouldn't it be great to search for keywords like Zanzibar, beach or sea and get those as a result?  That is called image search and it pretty much what I will try to build. 

For now I will follow the holy **KISS principle** and start with a fairly easy approach when it comes to the AI part:

- We extract the image category with a basic model trained with Imagenet. 
- We add the image to the predicted category in an inverted index. 
- We then use that database to retrieve all the images containing that category.

This framework allows us to include new models like an Object detection one (to detect more things) or more classification ones like Places365. Nevertheless, the described approach has some problems too:
- It lacks sense of synonyms/near-synonyms.
- We will be subject to the words we can predict. We will not be able to return more categories than those ones.

As a quick solution, we could create a synonyms dictionary to enlarge our categories but it would get messy to scale it. For now, I will keep it simple even though the existing problems.

So, as a next step for this functionality (and as a way to solve this problems), I have already thought about what it could come next to improve it and it's very exciting. A hint: it is related to <a href="https://github.com/moein-shariatnia/OpenAI-CLIP">this</a>.

### Similarity search

This feature will allow us to search by similarity by uploading an image. It is going to be machine-learning based and will look like this:

- We extract the image features with a basic model trained with Imagenet. 
- We reduce the features using PCA to remove not so releveant dimensions and reduce memory requirements. 
- We save and use them to build a multi-dimensional efficent query index using NMSLIB.
- We use that index to retrieve the k most similar images using k-nearest neighbour and a distance threshold.  


Moreover, I'll probably experiment with different ways to extract representations of the images like dhash or phash (which could be an option to select for the user if he wants to reduce time and memory requirements).
