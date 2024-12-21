```python
import asyncio
from decimal import Decimal
import time
from django.http import JsonResponse
from django.contrib.auth.decorators import login_required
from django.shortcuts import render
from django.utils import timezone
from patriot_app.async_tasks.async_utility import generate_pdf
from patriot_app.flags import ACTIVE, PURCHASE_SEND, SEND
from patriot_app.models import ContactsAddress, POHeader, ProformaDetail, ProformaHeader
from django.db.models import QuerySet, Max
from asgiref.sync import sync_to_async
from typing import List, Union, Tuple, Dict
from django.template.loader import render_to_string

from patriot_app.utils import user_log_create
from patriot_app.views import proforma_pdf_content, purchase_order_pdf_contect


async def async_download_proforma_to_customer(request, proforma_id):
    is_authenticated = await sync_to_async(lambda: request.user.is_authenticated)()
    if not is_authenticated:
        return JsonResponse({"error": "User not authenticated"}, status=400)

    try:
        proforma_header = await ProformaHeader.objects.aget(id=proforma_id)
    except ProformaHeader.DoesNotExist:
        return JsonResponse({"error": "Proforma not found"}, status=400)

    @sync_to_async
    def get_proforma_details() -> QuerySet[ProformaDetail]:
        return ProformaDetail.objects.filter(proformaheader=proforma_header,status=ACTIVE).order_by('lineno')
    @sync_to_async
    def filter_details_count(queryset:QuerySet[ProformaDetail]):
        return queryset.count()
    proforma_details = await get_proforma_details()
    if await filter_details_count(proforma_details) == 0:
        return JsonResponse({'msg': "Please add Items to proforma"}, status=400)
    
    if not proforma_header.confirmed_this_order:
        if not proforma_header.print_order_confirmed:
            proforma_header.print_order_confirmed = True
            proforma_header.sentdate = timezone.now()
            proforma_header.proforma_status = SEND
            proforma_header.asave()

    context = await sync_to_async(proforma_pdf_content)(proforma_header, proforma_details)
    html_content = await sync_to_async(render_to_string)('pshome/proforma/more-option/download.html', context)
    pdf_filename = 'proforma-confirmation'
    pdf_file_url_or_error, gen_status = await generate_pdf(html_content,pdf_filename)
    if gen_status == 400:
        return JsonResponse({'msg':pdf_file_url_or_error}, status=400)
    await sync_to_async(user_log_create)(request, request.user, action="Created", action_message="Proforma Printed confirmed Pdf", module_name='Proforma', module_instance_id=proforma_header.id)   
    return JsonResponse({'file_url':pdf_file_url_or_error,"filename":pdf_filename}, status=200)




async def async_download_purchase_order_report(request, purchase_order_id):
    is_authenticated = await sync_to_async(lambda: request.user.is_authenticated)()
    if not is_authenticated:
        return JsonResponse({"error": "User not authenticated"}, status=400)
    try:
        purchase_order = await POHeader.objects.aget(id=purchase_order_id)
    except POHeader.DoesNotExist:
        return JsonResponse({"error": "Purchase Order not found"}, status=400)

    if not purchase_order.is_downloded:
        purchase_order.is_downloded = True
        if not purchase_order.po_status != PURCHASE_SEND:
            purchase_order.po_status = PURCHASE_SEND # send flag for purchase order
            purchase_order.sentdate = timezone.now()
        purchase_order.asave()
    context = await sync_to_async(purchase_order_pdf_contect)(purchase_order)
    html_content = await sync_to_async(render_to_string)('pshome/purchase_order/more-option/view-po-doc.html', context)

    pdf_filename = 'purchase-order-report'
    pdf_file_url_or_error, gen_status = await generate_pdf(html_content,pdf_filename)
    if gen_status == 400:
        return JsonResponse({'msg':pdf_file_url_or_error}, status=400)

    await sync_to_async(user_log_create)(request, request.user, action="Download", action_message=f"{purchase_order.po_number} purchase order is Download Pdf", module_name='purchase_order', module_instance_id=purchase_order.id)   
    return JsonResponse({'file_url':pdf_file_url_or_error,"filename":pdf_filename}, status=200)

```
