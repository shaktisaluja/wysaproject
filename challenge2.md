#Challenge -2
## leetcode Problem-https://leetcode.com/problems/validate-binary-search-tree/ 

var isValidBST = function(root) {
    function validBST(node, left, right) {
        if(!node) return true;
        if(!(left < node.val && node.val < right)) return false;
        return validBST(node.left, left, node.val) && validBST(node.right, node.val, right);
    }
    return validBST(root, -Infinity, Infinity);
}