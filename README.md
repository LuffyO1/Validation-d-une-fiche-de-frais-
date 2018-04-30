# Validation-d-une-fiche-de-frais
function majFicheFrais($pdo)
{
    $lesFichesFrais = getLesFichesFrais($pdo);
    foreach ($lesFichesFrais as $uneFicheFrais) {
        $idVisiteur = $uneFicheFrais['idvisiteur'];
        $mois = $uneFicheFrais['mois'];
        $req = 'select sum(montant) as cumul from lignefraishorsforfait '
            . "where lignefraishorsforfait.idvisiteur = '$idVisiteur' "
            . "and lignefraishorsforfait.mois = '$mois' ";
        $res = $pdo->query($req);
        $ligne = $res->fetch();
        $cumulMontantHF = $ligne['cumul'];
        $req = 'select sum(lignefraisforfait.quantite * fraisforfait.montant) '
                . 'as cumul '
                . 'from lignefraisforfait, fraisforfait '
                . 'where lignefraisforfait.idfraisforfait = fraisforfait.id '
                . "and lignefraisforfait.idvisiteur = '$idVisiteur' "
                . "and lignefraisforfait.mois = '$mois' ";
        $res = $pdo->query($req);
        $ligne = $res->fetch();
        $cumulMontantForfait = $ligne['cumul'];
        $montantEngage = $cumulMontantHF + $cumulMontantForfait;
        $etat = $uneFicheFrais['idetat'];
        if ($etat == 'CR') {
            $montantValide = 0;
        } else {
            $montantValide = $montantEngage * rand(80, 100) / 100;
        }
        $req = "update fichefrais set montantvalide = $montantValide "
            . "where idvisiteur = '$idVisiteur' and mois = '$mois'";
        $pdo->exec($req);
    }
}
