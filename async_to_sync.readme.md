

```python
path('proforma/download/<uuid:proforma_id>/', download_proforma_to_customer, name="download_proforma_to_customer"),
```

## sync funtion
```python
@login_required(login_url='patriot_app:user_login')
def download_proforma_to_customer(request, proforma_id):
    proforma_header = ProformaHeader.objects.filter(id= proforma_id).first()
    proforma_details = ProformaDetail.objects.filter(proformaheader=proforma_header,status=ACTIVE).order_by('lineno')
    
    if proforma_details.count() == 0:
        return JsonResponse({'msg': "Please add Items to proforma"}, status=400)

    context = proforma_pdf_content(proforma_header, proforma_details)
    template = 'pshome/proforma/more-option/download.html'
    if not proforma_header.confirmed_this_order:
        if not proforma_header.print_order_confirmed:
            proforma_header.print_order_confirmed = True
            proforma_header.sentdate = timezone.now()
            proforma_header.proforma_status = SEND
            proforma_header.save()

    from asgiref.sync import async_to_sync
    from patriot_app.async_tasks.async_utility import generate_pdf
    
    pdf_filename = 'proforma-confirmation.pdf'
    html_content = render_to_string(template, context)
    pdf_file_url_or_error, gen_status = async_to_sync(generate_pdf)(html_content,pdf_filename)
    if gen_status == 400:
        return JsonResponse({'msg':pdf_file_url_or_error}, status=400)
    user_log_create(request, request.user, action="Created", action_message="Proforma Printed confirmed Pdf", module_name='Proforma', module_instance_id=proforma_header.id)   
    return JsonResponse({'file_url':pdf_file_url_or_error,"filename":pdf_filename}, status=200)

```

## async funtion
```python

from pyppeteer import launch
import os
from typing import List, Union, Tuple, Dict

from PATRIOT import settings
from patriot_app.s3 import upload_file_to_s3
from patriot_app.utils import delete_file_in_directory
from asgiref.sync import sync_to_async

async def generate_pdf(html_content, pdf_name) -> Tuple[str,int]:
    try:
        os.makedirs("media/", exist_ok=True)
        output_file_path = f'media/{pdf_name}'

        background_css = """
            @page {
                background-color: #FFFFFF;
                margin: 0;
            }
        """

        browser = await launch(
            handleSIGINT=False,
            handleSIGTERM=False,
            handleSIGHUP=False,
            headless=True,
            args=['--no-sandbox', '--disable-dev-shm-usage']
        )

        page = await browser.newPage()
        await page.setContent(html_content)
        await page.addStyleTag(content=background_css)

        await page.pdf({
            'path': output_file_path,
            'format': 'Letter',
            'printBackground': True,
            'margin': {
                'top': '0in',
                'right': '0in',
                'bottom': '0in',
                'left': '0in'
            }
        })

        await browser.close()
        await sync_to_async(upload_file_to_s3)(f"pdf/{pdf_name}", output_file_path)
        await sync_to_async(delete_file_in_directory)(output_file_path)        
        return f"{settings.BASE_IMAGE_URL}/pdf/{pdf_name}", 200   # Return the file path of the generated PDF
    except Exception as e:
        print(f"Error generating PDF: {e}")
        raise


```
