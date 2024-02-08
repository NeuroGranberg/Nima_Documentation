# Data Infrastructure and Workflow

## Infrastructure Components

   - Sectra PACS: Stores and manages medical images, receiving data directly from the MRI scanner.
   - NAS "Quantum": A storage solution with limited network file sharing and automatic data anonymization.
   - "Forskningsdator" or "Fractal Grande": The main server for data analysis, connected to the network with necessary data sharing protocols.

## Network Access

   - Tailscale: Ensures secure network access through a mesh network, requiring device registration.
   - SSH Access: Provides secure, remote server access for authorized users.
   - Portainer: Simplifies Docker container management, facilitating secure application deployment.
