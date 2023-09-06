# Python 서버

## 환경 변수 설정

  1) sysdm.cpl ,3 (실행 창에서 환경변수 창 열기)
  2) C:\Users\Hong Jun\AppData\Roaming\Python\Python39\Scripts 추가

## setup

  1) pip install pipenv
  2) pipenv --three => create the bubble
  3) pipenv shell => It's to go inside the bubble
        1) exit => deactivate shell
            2) deactivate => exit이 안 되면 deactivate
  4) pipenv install Django==2.2.5
  5) install python extention
  6) select the python interpreter
     1) ctrl+shift+p => python: select interpreter

## django project setup

  1) django-admin startproject config
  2) pipenv install flake8 --dev --pre
  3) select the linter in command palette
  4) pipenv install black --dev --pre (reference: python pep) 
  5) E501 line too long
     1) "python.linting.flake8Args":["--max-line-length=88"]
  6) Amend TIME_ZONE in settings.py
     1) UTC to Asia/Seoul
  7) pipenv install django-dotenv
     1) os.environ.get("")
     2) dotenv.read_dotenv() on if __name__ == "__main__" in manage.py

## django formatter

  1) Install extension
     1) Unibeautify
     2) Django
     3) Python

python manage.py makemigrations # Django will check my models.py and create migration files
python manage.py migrate # Whenever data type is changed, we need to migrate
python manage.py runserver
python manage.py createsuperuser

## django create apps

  1) django-admin startapp rooms

## settings.py

  1) INSTALLED_APPS = DJANGO_APPS + PROJECT_APPS
  2) AUTH_USER_MODEL = "user.User"
  3) PROJECT_APPS=["users.apps.UserConfig"]
-models.py
  1) class User(AbstractUser)
     1) model 작성[CHOICES, models.*Field]
     2) Pillow - python imagefiled handler
        1) pipenv install Pillow
      -admin.py
        @admin.register(models.User)
        class CustomUserAdmin(admin.ModelAdmin => UserAdmin) => models.py에 있는 model을 가져와서 admin panel에 추가한다.
           list_display
           list_filter
           fieldsets
        *UserAdmin => django에 있는 UserAdmin panel을 가져온다.

## core

class TimeStampedModel(models.Model)
   craeted auto_now_add
   updated auto_now - DateField
   class Meta:
       abstract=True

==apps==
-model
  models.Model / core model
  each fields, ForeignKey and ManyToManyField()
-admin
@admin.register(models.<class name>)
admin.ModelAdmin

### Room

1. Model 작성
  1) from django.db import models
  2) fields 작성
  3) foreign key or ManyToMany
  4) class Meta, STR
  5) AbstractItem()
  6) __str__
  7) __init__
  8) RomType - ForeignKey(on_delete=models.SET_NULL)
  9) django-countries
  10) Photo
       1) ImageField - upload_to / MEDIA_ROOT & MEDIA_URL in settings.py
       2) urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT) in urls.py
  11) save()
  12) from django.core.validators import MinValueValidator
       1) models.IntegerField(validator=[MinValueValidator(0)])
2. Admin 작성
   1) list_display
   2) PhotoInline(admin.TabularInline)
       1) model = mdoels.photo
   3) list_display, list_filter, search_fields, fieldsets, raw_id_fields, filter_horizontal, inlines, def.short_description
       1) icontains means insensitive contain on search_fields
   4) Photo - mark_safe

### Review 등등

### commands

  1) apps/management/commands/command_name.py
  2) help, add_arguments(), handle()
  3) django-seed, seeder.faker.address()
  4) lambda
  5) flatten
  6) datetime.datetime, timedelta - reservations

## url

  1) urlpatterns - path, include in config/urls.py
  2) "DIRS": [os.path.join(BASE_DIR, "templates")] in config/settings.py
       or "DIRS": [BASE_DIR / "templates"],
  3) room_views.HomeView.as_view(),

## views

  1) render, context
  2) Paginator
  3) except EmptyPage
  4) ListView - model, paginate_by, paginate_orphans, ordering, context_object_name
  5) pk
  6) DetailView
  7) SearchView - objects.filter, field lookups, order_by, getlist, filter_args={}
  8) View - get, SearchForm, form.is_valid,
  9) FormView - template_name, form_class, success_url, form_valid, authenticate, login
  10) reverse_lazy
  11) complete_verification
  12) SignUpForm(UserCreationForm)
  13) def get_form()
  14) router protection - UserPassesTestMixin
  15) @login_required & LOGIN_URL in settings.py
  16) models.Photo.objects.filter(pk=photo_pk).delete()
  17) hosting - request.session['hosting'] = True
  18) Class-Based Views REF : http://ccbv.co.uk/

### templates

  1) extends
  2) include
  3) block content
  4) &copy;
  5) template tags - {{page|add:-1}}
  6) Paginator - object_list, has_previous, number, paginator.num_pages, has_next, previous_page_number
  7) POST - csrf_token
  8) |slugify # int to text
  9) Reference : Context Processors
  10) |pluralize
  11) {{h_and_w|default:'h-20 w-20'}}
  12) date
  13) get_<status>_display

## model

  1) get_absolute_url
  2) verify_email, uuid, send_mail, 

### forms.py

  1) forms.Form - ModelChoiceField(queryset), ModelMultipleChoiceField, 
  2) django_countries.fields - CountryField()
  3) widget
  4) clean_email - cleaned_data, user.check_password, cleaned_dta, authenticate
  5) objects.create_user
  6) forms.From -> forms.ModelForm
  7) user.set_password

## mailgun.com

  1) sending -> domain setting -> smtp credentials
  2) settings.py
     1) EMAIL_HOST, EMAIL_PORT, EMAIL_HOST_USER, EMAIL_HOST_PASSWORD 
     2) You can find above things on SMTP credentials
  3) Create new SMTP user to get password
  4) Overview
     1) Authorize your email
  5) models.py
     1) email_verified, email_secret, def verify_email()
     2) strip_tags, render_to_string
  6) verify_email.html /users/verify/{secret}
  7) urls.py
     1) verify/<str:key>
  8) views.py - complete_verification
     1) email_verified = True, email_secret="", user.save() / success message, error message

## github login

  1) install requests
  2) models.py - login_method
  3) views.py - GithubException(Exception)
  4) scope means it requires user's information to login with github accout
  5) set_unusable_password

## kakao login

  1) Kakao developer
     1) User Management
     2) Redirect URI
  2) ContentFile from django.core.files.base

## tailwind CSS

  1) .gitignore
     1) uploads/
     2) node_modeuls/
  2) babel
     1) install @babel/core @babel/preset-env @babel/register
        (npm i @babel/{core,preset-env,register})
  3) gulp
     1) npm init
     2) scss : gulp, gulp-sass, gulp-postcss, gulp-csso, node-sass, autoprefixer, del
     3) js : babelify, gulp-bro
  4) install tailwindcss
     1) npm i tailwindcss
     2) npx tailwind init
  5) @tailwind * on styles.scss
  6) settings.py
     1) STATICFILES_DIRS = [os.path.join(BASE_DIR, "static")]
  7) <link rel="stylesheet" href="{% static 'css/styles.css' %}"> in .html
  8) install Tailwind CSS IntelliSence Extension
  9) @apply
  10) Customizing
      1) extend : { spacing : { "25vh":"25vh" } }
  11) room_card - truncate

## messages

  1) from django.contrib import messages

## form

  1) enctype = "multipart/form-data"
  2) request.FILES.get("name")

## templatetags (Custom template filter)

  1) register = template.Library
  2) @register.filter, @register.simple_tag (takes_context=True)
  3) {% load <.py name> %} in .html

## translator

  1) Add LOCALE_PATHS=  (BASE_DIR / "locale",) to settings.py
  2) create folder 'locale'
  3) {% load i18n %} in templates and {% trans "plaskjdf" %}, {% blocktrans %}
  4) Install extension(gettext)
  5) command
     1) django-admin makemessages --locale=ko
     2) django-admin compilemessages
  6) If you crash with erros and your os is window, you need to download gettext binary.
     Or OS is Mac, you need to execute 'brew link gettext --force' 
  7) Add django.middleware.locale.LocaleMiddleware to MIDDLEWARE on settings.py
  8) from django.utils import translation.LANGUAGE_SESSION_KEY on users/views.py
     1) 8번이 안 될 경우 LANGUAGE_COOKIE_NAME="django_language"를 settings.py에 추가한다.
     2) views.py 에서는 response.set_cookie(settings.LANGUAGE_COOKIE_NAME, lang)로 할당한다.
  9) {% get_current_language as LANGUAGE_CODE %} in template
  10)  Use 'from django.utils.translation import gettext_lazy as _' when you translate python code

## conversations

  1) from django.db.models import Q

연결 : ssh -i ./pem/django.pem ubuntu@ec2-52-79-236-230.ap-northeast-2.compute.amazonaws.com
-AWS
  1) Craete EC2 and Add pem/ at .gitignore
  2) DB - postgresql
     1) DEBUG = bool(os.environ.get("DEBUG"))
     2) Create db in aws
         1) DB instance identifier : RDS_NAME
         2) master name : RDS_USER
         3) password : RDS_PASSWORD
     3) RDS_HOST, NAME, USER, PASSWORD in settings.py
     4) *ubuntu settup
         1) psql -h <RDS_HOST> -p 5432 --username <RDS_USER_NAME> --dbname postgres
         2) CREATE DATABASE <RDS_NAME>;
  3) S3
     1) Create s3 on aws
     2) pipenv install django-storages
     3) pipenv install boto3
     4) Set the configuration for django-storages at settings.py
     5) create custom_sotrages.py on config folder
     6) STATICFILES_DIRS (on DEBUG)와 STATIC_ROOT 나누기
     6) *ubuntu settup
        1) Execute the command $ python manage.py collectstatic
            (This command will be executed on aws)
  4) Network
     1) ALLOWED_HOSTS=["*"]
  5) Social login
     1) users/views.py에 DEBUG 추가
     2) DEBUG에 따른 redirect_uri 구분
     3) Github 개발 OAuth 설정과 kakao developer 설정에서 uri 수정
  6) Create requirements.txt on developement environment
     1) pip freeze > requirements.txt
  7) ubuntu settup
     1) $ git clone <address>
     2) $ sudo apt update
     3) $ sudo apt install build-essential -y
     4) $ sudo apt install python3-pip -y
     5) $ pip3 install --upgrade pip
     6) $ sudo apt install virtualenv -y
     7) $ virtualenv -p python3 venv
     8) $ source venv/bin/activate
     9) $ pip install -r requirements.txt
     10) $ sudo apt install python3-psycopg2 -y
     11) $ sudo apt install postgresql-client -y
     13) $ pip install psycopg2-binary (https://stackoverflow.com/questions/8237842/django-core-exceptions-improperlyconfigured-error-loading-psycopg-module-no-mo)
     14) $ psql -h <RDS_HOST> -p 5432 --username <RDS_USER_NAME> --dbname postgres
     15) postgres=> CREATE DATABASE <RDS_NAME>;
     16) postgres=> exit
     17) $ python3 manage.py migrate
     18) $ python3 manage.py collectstatic
     19) $ sudo apt install gettext -y
     20) $ django-admin makemessages --locale=ko
     21) $ django-admin compilemessages
     -uwsgi
     22) $ pip install uwsgi
     23) $ vi uwsgi.ini
	[uwsgi]
	chdir=/home/ubuntu/{프로젝트 폴더}
	module={프로젝트 내 파일이름=config}.wsgi:application
	master=True
	pidfile=/tmp/project-master.pid
	vacuum=True
	max-requests=5000
	daemonize=/home/ubuntu/{프로젝트 폴더}/django.log
	home=/home/ubuntu/{프로젝트 폴더}/venv
	virtualenv=/home/ubuntu/{프로젝트 폴더}/venv
	socket=/home/ubuntu/{프로젝트 폴더}/uwsgi.sock
	chmod-socket=666
     24) $ uwsgi --ini uwsgi.ini
     25) $ sudo apt install nginx -y
     26) $ sudo vim /etc/nginx/nginx.conf

 	http{
 		upstream django{
 			server unix:/home/ubuntu/<project folder name>/uwsgi.sock;
 		}
 	}
 	 27) $ sudo vim /etc/nginx/sites-enabled/default
 	location / {
 		try_files $uri $uri/ =404; (이 부분 삭제)
 		include /etc/nginx/uwsgi_params;
 		uwsgi_pass django;
 	}
 	
 	location /static/ {
 		alias https://hairbnb3.s3.ap-northeast-2.amazonaws.com/static/;
 	}
 	
 	location /medai/ {
 		alias https://hairbnb3.s3.ap-northeast-2.amazonaws.com/uploads/;
 	}
 	 28)

https://www.youtube.com/watch?v=oGQ1HteFYnQ&t=762s