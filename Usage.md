# Usage #

## Quickstart ##

Add `sms` to your `INSTALLED_APPS` in `settings.py`:
```
    INSTALLED_APPS = (
        # apps
        'sms',
    )
```

Get in sync to update the database and install the provided `Carrier` fixtures:
```
    python manage.py syncdb
```

And play with some code:
```
    from django.contrib.auth.models import User
    from django.contrib.contenttypes.models import ContentType
    from sms.models import Carrier, ContentTypePhoneNumber, OutboundMessage
    from sms.util import send_sms

    # get a user
    matt = User.objects.get(username="matt")

    # get matt's cell phone carrier (at&t)
    att = Carrier.objects.get(pk=7)

    # attach a phone number to matt
    matts_phone_number, created = ContentTypePhoneNumber.objects.get_or_create(
        content_type = ContentType.objects.get_for_model(User),
        object_id = matt.pk,
        carrier = att,
        phone_number = "123-123-1234"
    )

    # do stuff (probably very important!!!!) ...

    # send matt an sms
    send_sms(
        msg = "omg im texting from python!",
        from_address = "mattdennewitz@gmail.com",
        recipient_list = [matts_phone_number],
        fail_silently = False
    )
```

## How Do I Make This Work? ##

SMS messages are sent with the `sms.util.send_sms` function,
which is nearly identical to `django.core.mail.send_mail`. The key differences are
that message subjects are not accepted and the `recipient_list` argument should be
a list of `ContentTypePhoneNumber` objects.

`django-sms` provides two three models for users to work with:

  1. `Carrier`, which represents the email-to-sms gateway provided by service providers. 61 carriers
> > are provided for you in the `fixtures`, which **should** cover most everyone's needs.


> 2. `ContentTypePhoneNumber`, which provides:

  * phone number storage
  * description of phone number - **optional**, meant for 'home' or 'work'
  * 'primary phone number' definition - **optional**, useful if you use this for contact storage

> `ContentTypePhoneNumber` also has a custom manager, `PhoneNumberManager`, which allows
> for `User` lookups via `ContentTypePhoneNumber.objects.get_for_user(user_obj)`. This manager
> will probably be fleshed out later, but thats it for now.

> 3. `OutboundMessage`, which acts as a message log. When a message is sent with `sms.util.send_sms`,
> > the `Carrier`, `ContentTypePhoneNumber` and dispatch time are logged. This is for pulling usage
> > information for both `Carrier`s and `ContentTypePhoneNumber`s.


> To pull data from the `OutboundMessage` logs, check out `OutboundMessageManager`. It has two helpers
> now, but will also probably be expanded in the future:

  * `most_popular_carriers`, which pulls the most popular carriers in a given (**optional**) date range
  * `most_contacted_numbers`, which pulls the most popular phone numbers is a given (**optional**) date range

> See their documentation for more.

## What's Coming ##

  * Signals. Maybe.
  * Forms, if people request such. Seems like a `ModelForm` for the `Carrier`s should suffice for now.
  * Requests?

## Words of Advice ##
  1. The data used in the fixtures was culled from Wikipedia. I didn't ask first, so I'm sorry for any toes I may have stepped on. Also, keep in mind that phone service providers are liable to shut off these services or make changes to format, rate limit, etc. on a whim and without notice. If you encounter any trouble, report them and we'll see what can happen.