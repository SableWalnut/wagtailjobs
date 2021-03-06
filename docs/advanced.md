#Customised job index page
`wagtailjobs` provides a default for the `jobs` index page which can be overridden by the user to display your custom fields, the default is
``` html
{% load i18n %}
<table class="listing full-width">
            <colgroup>
            <thead><tr class="table-headers">
                <th class="title">Name</th>
                <th class='issued_on'>Issued on</th>
                <th class="last_updated">Last Updated</th>
                <th class="date">job No.</th>
            </tr></thead>
            <tbody>
            {% for job in job_list|dictsortreversed:"issue_date" %}
                <tr>
                    <td class="title">
                        <h2>
                            <a href="{% url 'wagtailjobs_edit' pk=jobindex.pk job_pk=job.pk %}">
                                {% if not job.client_full_name %}{{job.client_organization}}{% else %}{{ job }}{% endif %}
                            </a>
                        </h2>
                        <ul class="actions">
                            <li><a href="{% url 'wagtailjobs_edit' pk=jobindex.pk job_pk=job.pk %}" class="button button-small">{% trans 'Edit' %}</a></li>
                            <li><a href="{% url 'wagtailjobs_delete' pk=jobindex.pk job_pk=job.pk %}" class="button button-small button-secondary no">{% trans 'Delete' %}</a></li>
                        </ul>
                    </td>
                    <td class='issued'> {{ job.issue_date|date:"d M Y H:i"}} </td>
                    <td class='updated'>
                        {% if job.last_updated %}
                        <div class="human-readable-date" title="{{ job.last_updated|date:"d M Y H:i" }}">
                        {{ job.last_updated|timesince }} ago</div>{% endif %}
                    </td>
                    <td class="date">
                        {{ job.id }}
                    </td>
                </tr>
            {% endfor %}
            </tbody>
</table>
```
You can define your own version of this inside `templates/wagtailjobs` as `job_list.html`
#Custom validation and overriding
Custom validation when saving an job can be done by defining a `validation.py` or similar, the name doesn't matter.
Custom validation is done in this way as a work around for wagtail `EditHandlers`.
Example usage
``` python
from django.contrib import messages
from wagtailjobs.utils.validation import validation as wagtailjobs_validation


def validation(request, job, is_sending_email):
    def is_job_address_fault():
        if job.client_organization or job.client_full_name:
            return False
        else:
            return True

    if is_job_address_fault() is True:
        messages.error(request, ('You cannot create an job without providing a client organization or client full name!'))
        return False

    else:
        return wagtailjobs_validation(request, job, is_sending_email)
```
To tell your project to override the default validation and include yours you will need to include this in your settings, for example
``` python
WAGTAIL_jobS_VALIDATION = 'ccc.joblist.validation.validation'
```
`WAGTAIL_jobS_VALIDATION` will need to be provided with the string representation of the path to your validation class.