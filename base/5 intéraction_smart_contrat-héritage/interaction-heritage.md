
# Interaction smart contrat  et héritage

Tu vas maintenant travailler avec de multiples smart contract. Tout d'abord, tu appelles des fonctions de smart contract à partir d'un autre smart contract. Ensuite, tu réalises héritage du comportement d'un smart contract parent dans un smart contract enfant.

Tout d'abord, tu remanies le code de l'exemple précédent pour créer un smart contract à jeton qui est distinct du code de MyContrat. Il s'agira d'un pseudo jeton ERC20, qui ne contiendra pas toutes les fonctionnalités d'un vrai smart contract ERC-20.

```
contract ERC20Token {
    string name;
    mapping(address => uint256) public balances;

    function mint() public {
        balances[msg.sender] += 1;
    }
}
```

Cela déplace la fonction mint(), le nom et la correspondance des équilibres vers le contrat symbolique puisque c'est là que ces responsabilités devraient se situer. Tu "frappes" des jetons à partir de MyContract en utilisant le contrat ERC20Token. Pour ce faire, tu as besoin de plusieurs éléments : l'adresse du jeton et le code du smart contrat pour appeler sa fonction mint(). D'abord stocke l'adresse du jeton dans une variable d'état comme celle-ci :


contract MyContract {
    address public token;
    //...
}

Tu peux maintenant définir la valeur de l'adresse du jeton dans le constructeur de la manière.

```
constructor(adresse payable _wallet, ERC20Token _token) {
    wallet = _wallet ;
    token = _token ;
}
```

tu as l'adresse disponible, tu peux accéder au code du smart contrat  à l'intérieur de la fonction buyToken() comme ceci, et appeler la fonction mint() 

```
fonction buyToken() public payable {
    ERC20Token _token = ERC20Token(adresse(token)) ;
    _token.mint() ;
    wallet.transfer(msg.value) ;
}
```

Si tu appelles la fonction buyToken() depuis ton compte personnel tu n'auras pas frapper de jetons pour toi-même. Revoyons la fonction mint()

```
fonction mint() public {
    balances [msg.sender] += 1 ;
}
```

La raison pour laquelle il ne frappera pas de jetons pour toncompte est que msg.sender fait en fait référence à l'adresse de MyContract, qui appelle la fonction depuis sa fonction buyToken() ! Si tu veux frapper des jetons pour ton propre compte, tu dois utiliser msg.sender pour référencer le compte qui a initié la transaction sur la chaîne de blocage.

```
fonction mint() public {
    soldes [msg.sender] += 1 ;
}
```
Et voilà ! Tu frappes des jetons d'un autre smart contrat ! Tu devrais d'abord déployer le smart contrat. Pour obtenir ton adresse, puis l'inclure comme argument chaque fois que vous déployez le second smart contrat. Voici à quoi ton code doit ressembler.

```
contract ERC20Token {
    string name;
    mapping(address => uint256) public balances;

    function mint() public {
        balances[msg.sender] += 1;
    }
}

contract MyContract {
    address public token;

    address payable wallet;

    constructor(address payable _wallet, address _token) {
        wallet = _wallet;
        token = _token;
    }

    function buyToken() public payable {
        ERC20Token _token = ERC20Token(address(token));
        _token.mint();
        wallet.transfer(msg.value);
    }
}
```

Tu va voir d'autres façons de faire référence au contrat "Token Smart".

```
function buyToken() public payable {
    ERC20Token(address(token)).mint();
    wallet.transfer(msg.value);
}
```


Parlons maintenant d'héritage. La solidité vous permet de créer des contrats intelligents qui héritent les uns des autres. Créons un nouveau pseudo jeton appelé MyToken qui hérite de notre contrat intelligent original. Nous utiliserons l'héritage pour lui donner un nom, un symbole et une fonction unique. Nous pouvons hériter du contrat intelligent de cette manière :

contract MyToken is ERC20Token {
    // ...

}


Nous pouvons maintenant garder la trace du nom du jeton dans le contrat parental.

```
contract ERC20Token {
    string public name;
    mapping(address => uint256) public balances;

    constructor(string memory _name) {
        name = _name;
    }

    function mint() public {
        balances[msg.sender] ++;
    }
}
```

Nous pouvons passer outre le nom du jeton parent dans le constructeur de notre jeton enfant. Pendant que tu es ici, tu vas également créer un symbole pour le jeton enfant et le placer dans le constructeur. tu va voir comment passer outre le constructeur du contrat du jeton parent, tout en attribuant les nouvelles valeurs comme ceci :

```
contract MyToken is ERC20Token {
    string public symbol;

    constructor(
        string memory _name,
        string memory _symbol
    )
        ERC20Token(_name)
    {
        symbol = _symbol;
    }
}
```

Personnalise MyToken. Mets à jour la fonction mint() pour remplacer la fonctionnalité du jeton parent. Tout d'abord, crées un moyen de stocker l'adresse de tous les propriétaires de jeton. Tu vas également définir le nombre de propriétaires comme ceci :

```
contract MyToken is ERC20Token {
    address[] public owners;
    uint256 public ownerCount;
    //...
}

```


Mets à jour ces valeurs dans ta propre fonction mint() tout en préservant le comportement du contrat intelligent du jeton parent. 

```
function mint() public {
    super.mint();
    ownerCount ++;
    owners.push(msg.sender);
}

```


Génial ! tu créé ton propre jeton personnalisé qui hérite du jeton parent !


```
contract ERC20Token {
    string public name;
    mapping(address => uint256) public balances;

    constructor(string memory _name) {
        name = _name;
    }

    function mint() public virtual {
        balances[msg.sender] ++;
    }
}

contract MyToken is ERC20Token {
    string public symbol;
    address[] public owners;
    uint256 public ownerCount;

    constructor(
        string memory _name,
        string memory _symbol
    )
        ERC20Token(_name)
    public {
        symbol = _symbol;
    }

    function mint() public override {
        super.mint();
        ownerCount ++;
        owners.push(msg.sender);
    }

}
```


Et voilà tu sais créer une interaction avec un smart contrat parent et un autre enfant !