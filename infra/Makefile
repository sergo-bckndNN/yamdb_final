build:
	sudo docker compose up -d --build
	sudo docker compose exec web python manage.py migrate
	sudo docker compose exec web python manage.py collectstatic --no-input
	sudo docker compose exec web python manage.py createsuperuser
