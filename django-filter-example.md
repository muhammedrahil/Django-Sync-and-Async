```python

async def async_download_proforma_to_customer(request, proforma_id):
    is_authenticated = await sync_to_async(lambda: request.user.is_authenticated)()
    if not is_authenticated:
        return JsonResponse({"error": "User not authenticated"}, status=400)

    # fetch the method - 1    
    try:
        proforma_header = await sync_to_async(lambda: ProformaHeader.objects.get(id=proforma_id))()
    except ProformaHeader.DoesNotExist:
        return JsonResponse({"error": "Proforma not found"}, status=404)
    except Exception as e:
        return JsonResponse({"error": str(e)}, status=500)
    
    @sync_to_async
    def get_proforma_details():
        queryset = ProformaDetail.objects.filter(
                proformaheader=proforma_header,
                status=ACTIVE
            ).order_by('lineno')
        return queryset

    proforma_details = await get_proforma_details()

    @sync_to_async
    def filter_details_count(queryset):
        return queryset.count()
        
    filtered_details = await filter_details_count(proforma_details)
    print(filtered_details)
    return JsonResponse({"msg":"Hello"}, status=200)

```
