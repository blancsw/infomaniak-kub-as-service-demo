# Pull base image
FROM python:3.12

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set work directory
WORKDIR /app

# Install dependencies
RUN pip install fastapi uvicorn
# Copy project
COPY . /app/

# Expose the application's port
EXPOSE 8080

# Run Gunicorn
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080", "--workers", "4", "--no-access-log", "--timeout-keep-alive", "2"]