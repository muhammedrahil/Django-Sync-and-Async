```python


import asyncio
from decimal import Decimal
import time
from django.http import JsonResponse
from django.contrib.auth.decorators import login_required
from django.shortcuts import render
from django.utils import timezone
from patriot_app.flags import ACTIVE, SEND
from patriot_app.models import ContactsAddress, ProformaDetail, ProformaHeader
from django.db.models import QuerySet, Max
from asgiref.sync import sync_to_async
```

```python
async def async_download_proforma_to_customer(request, proforma_id):
    is_authenticated = await sync_to_async(lambda: request.user.is_authenticated)()
    if not is_authenticated:
        return JsonResponse({"error": "User not authenticated"}, status=400)
    
     # Fetch the ProformaHeader instance with error handling
    try:
        proforma_header = await sync_to_async(lambda: ProformaHeader.objects.get(id=proforma_id))()
    except ProformaHeader.DoesNotExist:
        return JsonResponse({"error": "Proforma not found"}, status=404)
    except Exception as e:
        return JsonResponse({"error": str(e)}, status=500)
    
    print(proforma_header)
    return JsonResponse({"msg":"Hello"}, status=200)

```


```python
async def async_download_proforma_to_customer(request, proforma_id):
    is_authenticated = await sync_to_async(lambda: request.user.is_authenticated)()
    if not is_authenticated:
        return JsonResponse({"error": "User not authenticated"}, status=400)
    
     # Fetch the ProformaHeader instance with error handling
    proforma_header = await sync_to_async(ProformaHeader.objects.get)(id=proforma_id)
    print(proforma_header)
    return JsonResponse({"msg":"Hello"}, status=200)
```


```python

async def async_download_proforma_to_customer(request, proforma_id):
    is_authenticated = await sync_to_async(lambda: request.user.is_authenticated)()
    if not is_authenticated:
        return JsonResponse({"error": "User not authenticated"}, status=400)
    
     # Fetch the ProformaHeader instance with error handling
    proforma_header = await ProformaHeader.objects.aget(id=proforma_id)
    print(proforma_header)
    return JsonResponse({"msg":"Hello"}, status=200)
```
