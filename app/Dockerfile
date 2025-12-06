# Use a small base image
FROM python:3.12-slim

# Create a non-root user
RUN useradd -m -u 1001 appuser

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY main.py .

# Change ownership so non-root can read/write
RUN chown -R appuser:appuser /app

# Switch to non-root
USER appuser

# Expose port
EXPOSE 5000

# Run the app
CMD ["python", "main.py"]
