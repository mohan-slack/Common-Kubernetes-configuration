Sure, here's the content formatted for a GitHub README.md file:

```markdown
# Adding New Worker Nodes to Azure Singapore Nonprod Cluster

**Notes:** Ensure the firewall is open before adding new workers to your cluster.

## Steps

1. **Clone the openshift-deploy repo**
   ```sh
   git clone ssh://git@10.163.61.244:7999/rtsre/openshift-deploy.git
   cd openshift-deploy
   ```

2. **Add DNS entry for `asgprholdosw008` and `asgprholdosw009`**

3. **Add new entry to `_worker_nodes` in `vars/azure_nonprod_vars.yml`**
   ```yaml
   ...
   _worker_nodes:
   - { 'name': 'asgprholdosw001', 'ip': '10.197.39.11' }
   - { 'name': 'asgprholdosw002', 'ip': '10.197.39.12' }
   - { 'name': 'asgprholdosw003', 'ip': '10.197.39.13' }
   - { 'name': 'asgprholdosw004', 'ip': '10.197.39.14' }
   - { 'name': 'asgprholdosw005', 'ip': '10.197.39.15' }
   - { 'name': 'asgprholdosw006', 'ip': '10.197.39.16' }
   - { 'name': 'asgprholdosw007', 'ip': '10.197.39.17' }
   - { 'name': 'asgprholdosw008', 'ip': '10.197.39.18' }
   - { 'name': 'asgprholdosw009', 'ip': '10.197.39.19' }
   ...
   ```

4. **Run the wrapper script `run.sh`**
   ```sh
   ./run.sh -v vars/azure_nonprod_vars.yml -s vars/azure_nonprod_secrets.yml -r new_workers -a enterprise
   ```
   Enter new workers (comma separated): `asgprholdosw008,asgprholdosw009`

5. **Commit the updated var file**
   ```sh
   git commit -a -m "add 2 worker nodes to sg cluster dev"
   git push
   ```

Feel free to copy and paste this directly into your README.md file on GitHub! If you need any more adjustments, just let me know.
```
