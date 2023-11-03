#cd #!/bin/bash
#variables definides:
TFC='/sbin/iptables'

WEB='192.168.20.11'

LAN='enp0s8'
WAN='enp0s3'

#Funcio per habilitar el proxyfw com a passarel·la LAN-WAN:
habilitar_proxyfw()
{
       echo "Iniciant tallafoc..."

       # 1. Cridem a la funció buidar iptables per deixar el firewall sense regles.
            buidar_iptables
            
       # 2. Establim la política per defecte:
            $TFC -P INPUT DROP
            $TFC -P OUTPUT DROP
            $TFC -P FORWARD DROP

       # 3. Habilitem reenviament de paquets a nivell de kernel
            echo 1 > /proc/sys/net/ipv4/ip_forward

       # 4. Realitzem l'emmascarament dels paquets amb direcció WAN
            $TFC -t nat -A POSTROUTING -o $WAN -j MASQUERADE

}


#Funcio de regles finals per controlar determinat tipus de trànsit:
regles_alumne()
{
echo "afegint regles alumne..."

}

#Funcio per buidar totes les regles de iptables:
buidar_iptables()
{
for i in `cat /proc/net/ip_tables_names`; do
${TFC} -F -t $i
${TFC} -X -t $i
if [ $i = nat ]; then
        ${TFC} -t nat -P PREROUTING ACCEPT
        ${TFC} -t nat -P POSTROUTING ACCEPT
        ${TFC} -t nat -P OUTPUT ACCEPTelif [ $i = mangle ]; then
        ${TFC} -t mangle -P PREROUTING ACCEPT
        ${TFC} -t mangle -P INPUT ACCEPT
        ${TFC} -t mangle -P FORWARD ACCEPT
        ${TFC} -t mangle -P OUTPUT ACCEPT
        ${TFC} -t mangle -P POSTROUTING ACCEPT
elif [ $i = filter ]; then
        ${TFC} -t filter -P INPUT ACCEPT
        ${TFC} -t filter -P FORWARD ACCEPT
        ${TFC} -t filter -P OUTPUT ACCEPT
fi
done
        
}

# Funció que presenta el menu i permet escollir entre les opcions.
menu()
{
 ROOT_UID=0
 if [ $UID == $ROOT_UID ];
 then
        clear
        opcio=1
 while [ $opcio != 8 ]
     do
         echo -e "\t --- Menu - Tallafoc iptables en $HOSTNAME ---
                  CURS 2022-2023  S2
                  IP LAN:  $WEB

        1. Comprovar si iptables està instal·lat al sistema
        2. Buidar iptables
        3. Habilitar proxyfw
        4. Inserir regles alumne
        5. Mostrar les regles de iptables
        6. Fer backup de iptables en fitxer
        7. Restablir regles d' iptables des de fitxer
        8. Sortir \n"
        read opcio
          case $opcio in
                1) echo -e "Comprovant la instal·lació de iptables\n"
                        apt-cache policy iptables   ;;
        
                2) echo -e "Buidar regles del iptables\n"
                   buidar_iptables ;;
                    
                3)  echo -e "Habilitar proxyfw\n"
                    habilitar_proxyfw ;;
                
                4)  echo -e "Inserir regles alumne\n"
                    regles_alumne ;;

                5)  echo -e "Mostrant les regles del iptables\n"
                    $TFC -L -v -n ;;
                    
                6)  echo -e "Fer backup del iptables en fitxer\n"
                    echo "Escriu el nom del fitxer a guardar sense extensió"
                    read fitxer_regles
                    iptables-save > ${fitxer_regles}.fw
                    echo "S'ha guardat el fitxer amb nom" $fitxer_regles.fw ;;
                    
                7) echo -e "Restablir el i des de fitxer\n"
                    echo "Escriu el nom del fitxer a carregar sense extensió"
                    read fitxer_regles
                    iptables-restore  < ${fitxer_regles}.fw  ;;
                    
                8) exit 0 ;;
                
                *) echo -e "Opcio incorrecta!"
          esac
            # Fem una pausa i esborrem la pantalla
                        echo -e "\n\nPrem una tecla per continuar..."
                        read
                        clear
        done
else
 echo -e "Error: No t'has autenticat com a root per realitzar aquesta tasca"
fi
}

# ############# INICI DEL PROGRAMA PRINCIPAL  ##########################
# Després de definir totes les funcions comencem el programa principal
# cridant a la funcio menu per a que ens mostri el menú en pantalla:
menu

# Quan sortim del menú acabem el programa
exit 0

