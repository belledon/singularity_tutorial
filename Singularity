# Singularity
Bootstrap: docker
From: ubuntu:latest

%runscript
exec echo "The runscript is the containers default runtime command!"

%files
/home/crabby_patty/secret_recipe.txt /data/not_a_secret_recipe.txt

%environment
WHERESWALDO="/Ireland/Dublin/Merrion_Square_2011"
export WHERESWALDO

%labels
AUTHOR Thomas Anderson

%post
apt-get update 
apt-get -y install  python3 \
                    git \
                    wget
mkdir /data

%test
echo "Greetings, parental unit"
