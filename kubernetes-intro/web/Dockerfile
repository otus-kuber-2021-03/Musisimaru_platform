FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
RUN sed -i  "/^Listen 80.*/a Listen 8000" /usr/local/apache2/conf/httpd.conf
EXPOSE 8000