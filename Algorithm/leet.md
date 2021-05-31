##bit

-------------------------------

    判断是否为4的幂
    if( n<0 ||((n&(n-1))!=0)){
        return false;
    }
    return (n&0x55555555)==0?false:true;