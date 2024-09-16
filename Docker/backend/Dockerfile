# Dockerfile for NGINX Backend Server
FROM nginx:latest

# Copy the default NGINX configuration
COPY default.conf /etc/nginx/conf.d/default.conf

# Expose port 8080 for the backend servers
EXPOSE 8080

# Start NGINX in the foreground
CMD ["nginx", "-g", "daemon off;"]
