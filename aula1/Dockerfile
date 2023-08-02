# Use a imagem base do nginx
FROM nginx:latest

# Copie o arquivo de configuração personalizado para o contêiner
COPY nginx.conf /etc/nginx/nginx.conf

# Copie a página inicial personalizada para o contêiner
COPY index.html /usr/share/nginx/html

# Exponha a porta 80
EXPOSE 80

# Comando para iniciar o servidor nginx
CMD ["nginx", "-g", "daemon off;"]

