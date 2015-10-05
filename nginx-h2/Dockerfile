FROM ubuntu:14.04
MAINTAINER Bluesoft Fire <devops@bluesoft.com.br>

# Install dependencies
RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y install gcc g++ make patch wget libpcre3-dev zlib1g-dev libssl-dev

# Download nginx
RUN mkdir $HOME/src
RUN wget -O $HOME/src/nginx-1.9.3.tar.gz http://nginx.org/download/nginx-1.9.3.tar.gz 
RUN tar xzf $HOME/src/nginx-1.9.3.tar.gz -C $HOME/src/

# Apply http2 patch
RUN wget -O $HOME/src/nginx-1.9.3/patch.http2.txt http://nginx.org/patches/http2/patch.http2.txt
RUN cd $HOME/src/nginx-1.9.3/ && patch -p1 < patch.http2.txt

# Compile nginx
RUN mkdir -p /usr/local/nginx
RUN cd $HOME/src/nginx-1.9.3/ && ./configure --prefix=/usr/local/nginx/ --user=www-data --group=www-data --build=nginx-bsh2-1 --with-http_ssl_module --with-http_v2_module

RUN cd $HOME/src/nginx-1.9.3/ && make
RUN cd $HOME/src/nginx-1.9.3/ && make install

# Generate ssl key and certificate
RUN mkdir /usr/local/nginx/conf/ssl
RUN openssl genrsa -out /usr/local/nginx/conf/ssl/h2-server.key 2048
RUN openssl req -subj "/C=BR/ST=SP/L=São Paulo/O=Acme Inc/OU=DevOps" -new -key /usr/local/nginx/conf/ssl/h2-server.key -out /usr/local/nginx/conf/ssl/h2-server.csr
RUN openssl x509 -req -days 365 -in /usr/local/nginx/conf/ssl/h2-server.csr -signkey /usr/local/nginx/conf/ssl/h2-server.key -out /usr/local/nginx/conf/ssl/h2-server.crt

RUN rm /usr/local/nginx/conf/ssl/h2-server.csr
RUN chown -R www-data:www-data /usr/local/nginx/conf/ssl
RUN chmod 0400 /usr/local/nginx/conf/ssl/h2-server.key
RUN chmod 0444 /usr/local/nginx/conf/ssl/h2-server.crt

# Configure nginx
COPY nginx.conf.docker /usr/local/nginx/conf/nginx.conf
RUN mkdir -p /usr/local/nginx/html/

# Remove downloaded files
RUN rm -r $HOME/src/nginx-1.9.3
RUN rm $HOME/src/nginx-1.9.3.tar.gz

# Expose ports
EXPOSE 443

# Run nginx
ENTRYPOINT [ "/usr/local/nginx/sbin/nginx", "-g", "daemon off;" ]