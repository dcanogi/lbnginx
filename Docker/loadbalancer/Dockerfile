# Dockerfile for NGINX Load Balancer
FROM nginx:latest

# Copy the NGINX configuration file for the load balancer
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 8080 for the load balancer
EXPOSE 8080

# Start NGINX in the foreground
CMD ["nginx", "-g", "daemon off;"]
