
for djnago , give me urls, views like below template for "CustomUser" model

```py
from django.db import models

class Blog(models.Model):
    title = models.CharField(max_length=255)
    content = models.TextField()
    image = models.ImageField(upload_to='blog_images/', blank=True, null=True)

    def __str__(self):
        return self.title

```

**serserializers.py**
```py
# serializers.py
from rest_framework import serializers
from .models import Blog

class BlogSerializer(serializers.ModelSerializer):
    class Meta:
        model = Blog
        fields = ['id', 'title', 'content', 'image']

```


**urls**
```python
# urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('blogs/list/', api_blog_list),
    path('blogs/related/', api_blog_related),
    path('blogs/store/', api_blog_store'),
    path('blogs/<int:blog_id>/show/', api_blog_show),
    path('blogs/<int:blog_id>/edit/', api_blog_edit),
    path('blogs/<int:blog_id>/update/', api_blog_update),
    path('blogs/<int:blog_id>/destroy/', api_blog_destroy),
]

```

**views**
```python
# views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from ..models import *
from .serializers import *

@api_view(['GET'])
def api_blog_list(request):
    if request.method == 'GET':
        blogs = Blog.objects.all()
        serializer = BlogSerializer(blogs, many=True)
        return Response(serializer.data)
    
@api_view(['GET'])
def api_blog_related(request):
    # this is endpoint to get all foreign key field data for select input tag
    categories = Category.objects.all()
    brands = Brand.objects.all()

    category_serializer = CategorySerializer(categories, many=True)
    brand_serializer = BrandSerializer(brands, many=True)

    return Response({
        'categories': category_serializer.data,
        'brands': brand_serializer.data
    })

@api_view(['POST'])
def api_blog_store(request):
    serializer = BlogSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET'])
def api_blog_show(request, blog_id):
    try:
        blog = Blog.objects.get(id=blog_id)
        serializer = BlogSerializer(blog)
        return Response(serializer.data)
    except Blog.DoesNotExist:
        return Response({'message': 'Blog not found'}, status=status.HTTP_404_NOT_FOUND)

@api_view(['GET'])
def api_blog_edit(request, blog_id):
    try:
        blog = Blog.objects.get(id=blog_id)
        serializer = BlogSerializer(blog)
        return Response(serializer.data)
    except Blog.DoesNotExist:
        return Response({'message': 'Blog not found'}, status=status.HTTP_404_NOT_FOUND)

@api_view(['POST'])
def api_blog_update(request, blog_id):
    try:
        blog = Blog.objects.get(id=blog_id)
    except Blog.DoesNotExist:
        return Response({'message': 'Blog not found'}, status=status.HTTP_404_NOT_FOUND)

    serializer = BlogSerializer(blog, data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['POST'])
def api_blog_destroy(request, blog_id):
    try:
        blog = Blog.objects.get(id=blog_id)
    except Blog.DoesNotExist:
        return Response({'message': 'Blog not found'}, status=status.HTTP_404_NOT_FOUND)

    blog.delete()
    return Response({'message': 'Blog deleted successfully'}, status=status.HTTP_204_NO_CONTENT)

```