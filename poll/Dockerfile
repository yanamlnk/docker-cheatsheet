# poll/Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
ENV PORT=80
EXPOSE 80
CMD ["python", "app.py"]
