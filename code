// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title JeuDeDes
 * @dev Un jeu simple où les joueurs parient et lancent un dé
 * Si le résultat est 4, 5 ou 6, le joueur gagne le double de sa mise
 */
contract JeuDeDes {
    address public proprietaire;
    uint256 public montantMinimal = 0.001 ether;
    uint256 public cagnotte;
    
    // Événements pour suivre les parties
    event PartieLancee(address indexed joueur, uint256 mise, uint256 resultat, bool victoire, uint256 gain);
    event CagnotteReapprovisionee(address indexed donateur, uint256 montant);
    
    // Structure pour stocker l'historique d'un joueur
    struct Partie {
        uint256 mise;
        uint256 resultat;
        bool victoire;
        uint256 timestamp;
    }
    
    mapping(address => Partie[]) public historiqueJoueur;
    mapping(address => uint256) public gainsTotal;
    
    constructor() payable {
        proprietaire = msg.sender;
        cagnotte = msg.value;
    }
    
    modifier seulementProprietaire() {
        require(msg.sender == proprietaire, "Seul le proprietaire peut faire cela");
        _;
    }
    
    /**
     * @dev Lance le dé et détermine si le joueur gagne
     * Le joueur gagne si le résultat est >= 4 (4, 5 ou 6)
     */
    function lancerDe() external payable {
        require(msg.value >= montantMinimal, "Mise trop faible");
        require(cagnotte >= msg.value * 2, "Cagnotte insuffisante pour couvrir les gains potentiels");
        
        // Génération pseudo-aléatoire du résultat (1-6)
        // Note: En production, utiliser Chainlink VRF pour une vraie aléatoire
        uint256 resultat = (uint256(keccak256(abi.encodePacked(
            block.timestamp,
            block.prevrandao,
            msg.sender
        ))) % 6) + 1;
        
        bool victoire = resultat >= 4;
        uint256 gain = 0;
        
        if (victoire) {
            gain = msg.value * 2;
            cagnotte -= gain;
            payable(msg.sender).transfer(gain);
            gainsTotal[msg.sender] += gain;
        } else {
            cagnotte += msg.value;
        }
        
        // Enregistrer la partie dans l'historique
        historiqueJoueur[msg.sender].push(Partie({
            mise: msg.value,
            resultat: resultat,
            victoire: victoire,
            timestamp: block.timestamp
        }));
        
        emit PartieLancee(msg.sender, msg.value, resultat, victoire, gain);
    }
    
    /**
     * @dev Permet au propriétaire de réapprovisionner la cagnotte
     */
    function reapprovisionnerCagnotte() external payable seulementProprietaire {
        cagnotte += msg.value;
        emit CagnotteReapprovisionee(msg.sender, msg.value);
    }
    
    /**
     * @dev Retourne le nombre de parties jouées par un joueur
     */
    function obtenirNombreParties(address joueur) external view returns (uint256) {
        return historiqueJoueur[joueur].length;
    }
    
    /**
     * @dev Retourne une partie spécifique d'un joueur
     */
    function obtenirPartie(address joueur, uint256 index) external view returns (
        uint256 mise,
        uint256 resultat,
        bool victoire,
        uint256 timestamp
    ) {
        require(index < historiqueJoueur[joueur].length, "Index invalide");
        Partie memory p = historiqueJoueur[joueur][index];
        return (p.mise, p.resultat, p.victoire, p.timestamp);
    }
    
    /**
     * @dev Permet au propriétaire de retirer une partie de la cagnotte
     */
    function retirerCagnotte(uint256 montant) external seulementProprietaire {
        require(montant <= cagnotte, "Montant trop eleve");
        cagnotte -= montant;
        payable(proprietaire).transfer(montant);
    }
    
    /**
     * @dev Modifier la mise minimale
     */
    function changerMiseMinimale(uint256 nouvelleMise) external seulementProprietaire {
        montantMinimal = nouvelleMise;
    }
    
    /**
     * @dev Obtenir le solde du contrat
     */
    function obtenirSoldeContrat() external view returns (uint256) {
        return address(this).balance;
    }
}
