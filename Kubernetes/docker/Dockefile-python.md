# python

Here's a breakdown of what each part of your **Dockerfile** does:
``` Dockerfile
# Inherit from base Python image.

# FROM af.cds.bns:5003/distroless/python3:debug

FROM af.cds.bns:5003/python:3.12-bullseye

  

# Build-time arguments.

ARG CA_BUNDLE

ARG ARTIFACTORY_USERNAME_READONLY

ARG ARTIFACTORY_PASSWORD_READONLY

ARG USERNAME=pythonuser

ARG USERGROUP=pythonuser

ARG USERID=1004

ARG GROUPID=$USERID

  

# Create a non-root user.

RUN addgroup --gid $GROUPID --system $USERGROUP && \

adduser --uid $USERID --system $USERNAME --ingroup $USERGROUP --shell /bin/false

USER root

  

# Create the new working directory.

RUN mkdir -p /app

  

# Copy the locustfile.

COPY . .

# Copy the dependency list.

COPY requirements.txt /app/requirements.txt

  

# Set the working directory.

WORKDIR /app

  

# Install dependencies.

RUN echo "${CA_BUNDLE}" > /tmp/root.pem && \

python -m pip install \

--no-cache-dir \

--cert /tmp/root.pem \

-i "https://${ARTIFACTORY_USERNAME_READONLY}:${ARTIFACTORY_PASSWORD_READONLY}@af.cds.bns/artifactory/api/pypi/virtual-python-bns/simple" \

-r /app/requirements.txt && \

rm -rfv /usr/local/lib/python3.11/site-packages/gevent/tests && \

rm /tmp/root.pem

  

# Grant ownership to files for non-root user.

RUN chmod 755 /app/locustfile.py && chown -R $USERNAME:$USERGROUP /app

  

# Use the non-root user.

USER $USERNAME

  

EXPOSE 8089 8089

CMD ["locust", "-P", "8089", "-f", "locustfile.py"]
```

---

### 🐍 **Base Image**

```Dockerfile
FROM af.cds.bns:5003/python:3.12-bullseye
```

- Uses a **custom Python 3.12 image** based on Debian Bullseye from a private registry (`af.cds.bns:5003`).
- The commented-out line suggests an alternative **distroless** image for minimal containers.

---

### ⚙️ **Build-Time Arguments**

```Dockerfile
ARG CA_BUNDLE
ARG ARTIFACTORY_USERNAME_READONLY
ARG ARTIFACTORY_PASSWORD_READONLY
ARG USERNAME=pythonuser
ARG USERGROUP=pythonuser
ARG USERID=1004
ARG GROUPID=$USERID
```

- These are variables passed at build time.
- Includes credentials for a private Python package index and user/group info for creating a non-root user.

---

### 👤 **Create a Non-Root User**

```Dockerfile
RUN addgroup --gid $GROUPID --system $USERGROUP && \
    adduser --uid $USERID --system $USERNAME --ingroup $USERGROUP --shell /bin/false
```

- Adds a **system group and user** with no login shell for security.
- Ensures the container doesn't run as root.

---

### 📁 **Prepare Working Directory**

```Dockerfile
RUN mkdir -p /app
COPY . .
COPY requirements.txt /app/requirements.txt
WORKDIR /app
```

- Creates `/app` directory.
- Copies all files and `requirements.txt` into the container.
- Sets `/app` as the working directory.

---

### 📦 **Install Python Dependencies**

```Dockerfile
RUN echo "${CA_BUNDLE}" > /tmp/root.pem && \
    python -m pip install \
    --no-cache-dir \
    --cert /tmp/root.pem \
    -i "https://${ARTIFACTORY_USERNAME_READONLY}:${ARTIFACTORY_PASSWORD_READONLY}@af.cds.bns/artifactory/api/pypi/virtual-python-bns/simple" \
    -r /app/requirements.txt && \
    rm -rfv /usr/local/lib/python3.11/site-packages/gevent/tests && \
    rm /tmp/root.pem
```

- Uses a **custom CA certificate** to trust internal Artifactory.
- Installs dependencies from a **private PyPI repo**.
- Cleans up test files and the temporary cert after install.

---

### 🔐 **Permissions and Ownership**

```Dockerfile
RUN chmod 755 /app/locustfile.py && chown -R $USERNAME:$USERGROUP /app
```

- Ensures the `locustfile.py` is executable and owned by the non-root user.

---

### 🚀 **Run as Non-Root User**

```Dockerfile
USER $USERNAME
```

- Switches to the non-root user for security.

---

### 🌐 **Expose and Run Locust**

```Dockerfile
EXPOSE 8089 8089
CMD ["locust", "-P", "8089", "-f", "locustfile.py"]
```

- Exposes port 8089 (Locust web UI).
- Runs Locust using the specified file.

---

Would you like help converting this to use a `distroless` image or optimizing it further?