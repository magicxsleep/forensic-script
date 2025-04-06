# forensic-script

# Define que o script será executado no Bash.
#!/bin/bash

# Verifica se foi passado um arquivo como argumento
if [ -z "$1" ]; then
    echo "❌ Uso: $0 <arquivo>"
    exit 1
fi

# Verifica se o arquivo existe
if [ ! -f "$1" ]; then
    echo "❌ Arquivo não encontrado: $1"
    exit 1
fi

# Detecta o pendrive e sua partição
echo -e "\033[1;34m🔌 Conecte o PENDRIVE...\033[0m"
while true; do
    # Detecta dispositivos removíveis (RM=1) usando `lsblk`
    DISPOSITIVO=$(lsblk -dno NAME,RM | awk '$2==1 {print $1}' | head -n 1)
    if [ -n "$DISPOSITIVO" ]; then
        echo -e "\033[1;32m✅ Pendrive detectado: /dev/$DISPOSITIVO\033[0m"
        break
    fi
    sleep 1
done

# Verifica e monta automaticamente a partição correta
PARTICAO=$(lsblk -lnp -o NAME,RM | awk -v device="/dev/$DISPOSITIVO" '$1 ~ device {print $1}' | head -n 1)
if [ -z "$PARTICAO" ]; then
    echo -e "\033[1;31m❌ Nenhuma partição encontrada no pendrive!\033[0m"
    exit 1
fi
MOUNT_POINT="/mnt/pendrive"
mkdir -p "$MOUNT_POINT"
mount "$PARTICAO" "$MOUNT_POINT" || {
    echo -e "\033[1;31m❌ Falha ao montar o pendrive. Verifique o formato.\033[0m"
    exit 1
}

# Converte o arquivo para hexadecimal usando o Sleuth Kit
convert2hex=$(tsk_recover -e "$1" 2>/dev/null | xxd -p)
if [ -z "$convert2hex" ]; then
    echo "❌ Falha ao converter o arquivo para hexadecimal."
    umount "$MOUNT_POINT"
    exit 1
fi

# Remove espaços da sequência hexadecimal
result=$(echo "$convert2hex" | tr -d ' ')

# Aqui você pode adicionar mais comandos para usar a variável `result` conforme necessário

# Desmonta o pendrive
umount "$MOUNT_POINT"
echo -e "\033[1;32m✅ Operação concluída com sucesso!\033[0m"
