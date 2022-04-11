# Pagination in Django REST framework

Complete pagination in drf.

## Steps

### Set up a new virtual environment 

```
python3 -m venv env
source env/bin/activate

python3 -m pip install django
python3 -m pip install djangorestframework
python3 -m pip install mysql
python3 -m pip install pygments  # using this for the code highlighting
```

### Start a project

```
django-admin startproject tutorial
cd tutorial

python manage.py startapp snippets
```

### Configure database

```
source ~/.bash_profile
mysql -u root -p
create database Snippets
quit
```

### Edit the settings.py file

- INSTALLED_APPS

  ```python
  INSTALLED_APPS = [
      ...
      'rest_framework',
      'snippets',
  ]
  ```

- DATABASES

  ```python
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': 'Snippets',
          'HOST': '127.0.0.1',
          'PORT': '3306',
          'USER': 'root',
          'PASSWORD': 'password'
      }
  }
  ```

- TIME_ZONE

  ```python
  TIME_ZONE = 'Asia/Shanghai'
  ```

- REST_FRAMEWORK

  ```python
  REST_FRAMEWORK = {
      'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
      'PAGE_SIZE': 10,
      'MAX_PAGE_SIZE': 100,
  }
  ```

### Create a model

### Migrate

```
python manage.py makemigrations snippets
python manage.py migrate snippets
```

### Create a serializer class

### Write APIViews

### Pagination

```python
class StandardResultsSetPagination(PageNumberPagination):

    page_size = 3  # 每页显示的条数
    page_query_param = 'current'  # 前端发送的页数的关键字名
    page_size_query_param = 'pageSize'  # 前端发送的每页条数的关键字名
    max_page_size = 10  # 每页最大显示的条数


    def paginate_queryset(self, queryset, request, view=None):
        """
        Paginate a queryset if required, either returning a
        page object, or `None` if pagination is not configured for this view.
        """
        empty = True

        page_size = self.get_page_size(request)
        if not page_size:
            return None

        paginator = self.django_paginator_class(queryset, page_size)
        page_number = request.query_params.get(self.page_query_param, 1)
        if page_number in self.last_page_strings:
            page_number = paginator.num_pages

        try:
            self.page = paginator.page(page_number)
        except InvalidPage as exc:
            empty = False
            pass

        if paginator.num_pages > 1 and self.template is not None:
            self.display_page_controls = True

        self.request = request

        if not empty:
            self.page = []

        return list(self.page)
```

### Wire up views

```python
class SnippetList(APIView):
    """
    List all snippets, or create a new snippet.
    """
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        page = StandardResultsSetPagination()
        res = page.paginate_queryset(snippets, request)
        if res is not None:
            ser = SnippetSerializer(res, many=True)
            return Response(ser.data, status=status.HTTP_200_OK)
        else:
            return Response(status=status.HTTP_200_OK)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### Runserver

```
python manage.py runserver
```

