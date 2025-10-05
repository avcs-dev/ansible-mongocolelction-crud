# Ansible MongoDB Collection CRUD

This repository demonstrates how to consume the community MongoDB Ansible collection offline, install it, and use it in Ansible playbooks to perform CRUD operations on MongoDB.

## Prerequisites

- Ansible installed on your system.
- Python packages: `pymongo` (install with `pip install pymongo`).
- MongoDB shell: `mongosh` (required for `mongodb_shell` module).
- Access to MongoDB instance.
- Offline tarball files for the required collections (available in the `collections/` directory).

## Collections

The following collections are used:

- `community.mongodb` (version 1.7.10)
- `ansible.posix` (version 2.1.0)
- `community.general` (version 11.3.0)

The tarball files are located in the `collections/` directory:
- `community-mongodb-1.7.10.tar.gz`
- `ansible-posix-2.1.0.tar.gz`
- `community-general-11.3.0.tar.gz`

## Installation

### Offline Installation

1. **Download the Collections (on a machine with internet):**
   ```sh
   ansible-galaxy collection download community.mongodb
   ansible-galaxy collection download ansible.posix
   ansible-galaxy collection download community.general
   ```
   This creates tarball files like `community-mongodb-<version>.tar.gz`.

2. **Transfer the Tarballs:**
   Use `scp`, `rsync`, or a USB drive to move the `.tar.gz` files to your offline environment.

3. **Install from requirements.yml:**
   Navigate to the collections directory and install:
   ```sh
   cd collections
   ansible-galaxy collection install -r requirements.yml
   ```
   This installs the collections to the default Ansible collections path (e.g., `~/.ansible/collections/ansible_collections/`).

4. **Verify Installation:**
   ```sh
   ansible-galaxy collection list
   ```

### Custom Collection Path

To install to a custom path:
```sh
ansible-galaxy collection install -r requirements.yml --collections-path /path/to/custom/collections
```

Then, set the environment variable when running playbooks:
```sh
export ANSIBLE_COLLECTIONS_PATHS=/path/to/custom/collections
ansible-playbook mongodb_crud.yml
```

## Docker Setup

This project includes a `docker-compose.yaml` to run MongoDB in a container for testing.

1. **Start MongoDB:**
   ```sh
   sudo docker compose up -d
   ```
   - MongoDB runs with root user `myuser` and password `mypass`.
   - Accessible at `host.docker.internal:27017` from within the dev container.

2. **Stop MongoDB:**
   ```sh
   sudo docker compose down
   ```

Note: In dev containers, use `host.docker.internal` to connect to the host's Docker containers. For non-dev container setups, use `localhost` if MongoDB is running locally.

## Usage in Ansible Playbook

Reference the collection in your playbook and use the modules to interact with MongoDB.

### Example Playbook

Create a playbook file, e.g., `mongodb_crud.yml`:

```yaml
---
- hosts: localhost
  collections:
    - community.mongodb
  tasks:
    - name: Check MongoDB Status
      community.mongodb.mongodb_info:
        login_user: myuser
        login_password: mypass
        login_host: host.docker.internal  # Use localhost for non-Docker setups
        login_port: 27017
      register: mongo_info

    - debug:
        msg: "MongoDB is running. Version: {{ mongo_info.general.version }}"

    - name: Create a database and insert a document
      community.mongodb.mongodb_shell:
        login_user: myuser
        login_password: mypass
        login_host: host.docker.internal
        login_port: 27017
        db: mydb
        eval: |
          db.mycollection.insertOne({name: "example", value: 123})

    - name: Query documents
      community.mongodb.mongodb_shell:
        login_user: myuser
        login_password: mypass
        login_host: host.docker.internal
        login_port: 27017
        db: mydb
        eval: db.mycollection.find({}).toArray()
      register: result

    - debug:
        var: result

    - name: Update a document
      community.mongodb.mongodb_shell:
        login_user: myuser
        login_password: mypass
        login_host: host.docker.internal
        login_port: 27017
        db: mydb
        eval: |
          db.mycollection.updateOne({name: "example"}, {$set: {value: 456}})

    - name: Delete a document
      community.mongodb.mongodb_shell:
        login_user: myuser
        login_password: mypass
        login_host: host.docker.internal
        login_port: 27017
        db: mydb
        eval: |
          db.mycollection.deleteOne({name: "example"})
```

### Running the Playbook

```sh
ansible-playbook mongodb_crud.yml
```

For custom collection paths:
```sh
export ANSIBLE_COLLECTIONS_PATHS=/path/to/custom/collections
ansible-playbook mongodb_crud.yml
```

### Notes

- The `community.mongodb` collection uses `mongodb_shell` for executing MongoDB commands, as direct CRUD modules are not available.
- Ensure `pymongo` and `mongosh` are installed on the Ansible control node.
- For Docker setups in dev containers, use `host.docker.internal` as the host.
- For local MongoDB, use `localhost`.
- Adjust `login_host`, `login_user`, and `login_password` as needed for your MongoDB setup.

## Non-Docker Setup

For setups without Docker, ensure MongoDB is running locally or remotely.

1. **Install MongoDB locally** (e.g., on Ubuntu/Debian):
   ```sh
   sudo apt update
   sudo apt install mongodb
   sudo systemctl start mongodb
   ```

2. **Install Collections to Custom Path:**
   ```sh
   mkdir -p /opt/ansible/collections
   ansible-galaxy collection install -r collections/requirements.yml --collections-path /opt/ansible/collections
   ```

3. **Run Playbook with Custom Path:**
   ```sh
   export ANSIBLE_COLLECTIONS_PATHS=/opt/ansible/collections
   ansible-playbook mongodb_crud.yml
   ```

4. **Update Playbook Host:**
   Change `login_host: host.docker.internal` to `login_host: localhost` (or your MongoDB server's IP).

## Additional Resources

- [Community MongoDB Collection Documentation](https://docs.ansible.com/ansible/latest/collections/community/mongodb/)
- [Ansible Galaxy Collections](https://galaxy.ansible.com/)

## Contributing

Feel free to submit issues or pull requests.

## License

This project is licensed under the MIT License.

If you encounter issues (e.g., missing tarballs), ensure the files are present and the versions match. For online installation (if needed), remove the `.tar.gz` extensions and versions from requirements.yml, then run the command on a machine with internet access. Let me know if you need help modifying the file or troubleshooting.