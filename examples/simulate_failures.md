## Testes e Simulações

### Simulação de Falhas no Cluster

#### Teste 1: Falha de Nó
1. Identifique o nó a ser simulado como indisponível.
2. Execute o comando para drenar o nó no cluster Kubernetes:
   ```bash
   kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
   ```
3. Verifique se os pods são redistribuídos para outros nós:
   ```bash
   kubectl get pods -n gitlab -o wide
   ```
4. Após o teste, desmarque o nó:
   ```bash
   kubectl uncordon <node-name>
   ```

#### Teste 2: Falha de Rede
1. Simule uma falha de rede desconectando a interface de rede do nó principal:
   ```bash
   ip link set <interface-name> down
   ```
2. Verifique se os recursos do GitLab são migrados para outro nó.
3. Restaure a conectividade:
   ```bash
   ip link set <interface-name> up
   ```

#### Teste 3: Falha de Armazenamento
1. Simule uma falha no volume persistente desativando o Longhorn:
   ```bash
   kubectl scale deployment longhorn-manager --replicas=0 -n longhorn-system
   ```
2. Verifique se os serviços do GitLab entram em estado de erro.
3. Restaure o Longhorn:
   ```bash
   kubectl scale deployment longhorn-manager --replicas=1 -n longhorn-system
   ```