-- window : set timezone.
$env:MAVEN_OPTS="-Duser.timezone=Asia/Kolkata"
$env:PYTHONWARNINGS = "ignore"

-- run single profile
mvn clean compile flyway:migrate -Pdb1-local

-- repair flyway 
mvn flyway:repair -Pdb2-local

-- compare schema
migra --unsafe postgresql://airflow:airflow@localhost:5432/db2 postgresql://airflow:airflow@localhost:5432/db1



-- setup with jenkins.
1. Dockerfile
2. docker-compose.yml


-- jenkins script for poweshell .
#!/bin/bash
set -e

echo "--- 1. BUILDING AUTOMATION RUNNER IMAGE ---"
# Build image (uses current Dockerfile with 'COPY auto_migrate_schema.py /app/')
docker build --no-cache -t schema-sync-runner-final .

echo "--- 2. EXECUTING SCHEMA SYNCHRONIZATION ---"

# CRITICAL FIX: Use --entrypoint to tell Docker to execute /bin/bash 
# instead of the base image's default entrypoint script (mvn-entrypoint.sh).
docker run --rm \
  --entrypoint "/bin/bash" \
  --network=common-network \
  -v "$(pwd)":/app \
  schema-sync-runner-final \
  -c "python /app/auto_migrate_schema.py"

echo "--- 3. PIPELINE COMPLETE ---"