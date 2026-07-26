# pythonnurr — Build Notes

## Status
- Blueprint: fastapi-ec2@1.0.0 (monitoring=none)
- Meta approved: aws / us-east-1 / ec2 / t3.micro / github / repo=nurpython4
- Design confirmed ✅
- Plan approved ✅
- Generation complete — validate_project PASS (21 files)
- **Next: create_repo_and_push → deploy**

## Decisions
- No database (explicit user requirement)
- No monitoring (user preference, Tier 1)
- No load balancer (single instance, nginx proxy)
- Ansible playbook: copy src paths prefixed with {{ playbook_dir }}/../ (known-issue fix applied proactively)
- wait_for_connection + apt retries added for cold-boot robustness
- handlers used for idempotent systemd restarts
- wait_for port 8000 added before verify stage hits the health check

## Known-issue warnings assessed (validate_project)
- ansible.builtin.synchronize: not used — N/A
- ansible.posix.synchronize: not used — N/A
- copy src paths: FIXED ({{ playbook_dir }}/../ prefix applied)
- wait_for_connection /var/www/ perm: our app runs as root under /opt/app — N/A
- copy content Jinja2: our content blocks use nginx $vars not {{ }} — N/A

## Stack
- Python 3.11 / FastAPI / uvicorn / nginx / Ubuntu 22.04
- Terraform: infra/ (EC2, EIP, SG, key pair)
- Ansible: ansible/playbook.yml (single play, handlers)
- Pipeline: provision → configure → verify (self-sufficient jobs)
