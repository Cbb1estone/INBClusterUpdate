# Creación de Máquinas Virtuales en Azure

## Crear Máquina Virtual

### Turing
```bash
az vm create \
  --resource-group UNAM \
  --name Turing \
  --public-ip-sku Standard \
  --image Ubuntu2204 \
  --admin-username turing \
  --generate-ssh-keys \
  --vnet-name VM-Network \
  --subnet default
```

### Gates
```bash
az vm create \
  --resource-group UNAM \
  --name Gates \
  --public-ip-sku Standard \
  --image Ubuntu2204 \
  --admin-username gates \
  --generate-ssh-keys \
  --vnet-name VM-Network \
  --subnet default
```

### Hopper
```bash
az vm create \
  --resource-group UNAM \
  --name Hopper \
  --public-ip-sku Standard \
  --image Ubuntu2204 \
  --admin-username hopper \
  --generate-ssh-keys \
  --vnet-name VM-Network \
  --subnet default
```

## Ver la NSG (Network Security Group)
```bash
az network nsg list --resource-group UNAM --output table
```

## Abrir Puerto 22 (SSH)
```bash
az network nsg rule create \
  --resource-group UNAM \
  --nsg-name TuringNSG \
  --name AllowSSH \
  --priority 1000 \
  --protocol Tcp \
  --direction Inbound \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22 \
  --access Allow
```

## Ver IP de la VM
```bash
az vm list-ip-addresses --resource-group UNAM --output table
```

## Conectarse a la VM
```bash
ssh turing@<public-ip>
```

## Verificar claves SSH
```bash
ls -l ~/.ssh/
```

## Verificar el archivo de Public IP
```bash
ssh -i /ruta/a/tu/clave_privada.pem gates@52.255.202.84
```

---
*Última actualización: 12 de marzo de 2025*
