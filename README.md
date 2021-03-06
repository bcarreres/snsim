# Code for simulation of SN Ia using sn cosmo
## Installation
In the setup.py directory use:
```
>python -m pip setup .
```
## Input file :
The input file is a .yml with the following structure:
```
data :
    write_path: '/PATH/TO/OUTPUT'
    sim_name: 'NAME OF SIMULATION'
    band_dic: {'r':'ztfr','g':'ztfg','i':'ztfi'} #(Optional -> if bandname in db/obs file doesn't
 correpond to those in sncosmo registery)
    write_format: 'format' or ['format1','format2'] #(Optional default pkl, fits)
db_config: 
    dbfile_path: '/PATH/TO/FILE'
    add_keys: ['keys1', 'keys2', ...] #(Optional add db file keys to metadata)  
    db_cut: {'key1': ["conditon1","conditon2",...], 'key2':["conditon1"],...} #(Optional SQL conditions on key)
    zp: INSTRUMENTAL ZEROPOINT  
    ra_size: RA FIELD SIZE in DEG
    dec_size: DEC FIELD SIZE in DEG
    gain: CCD GAIN e-/ADU
sn_gen:
    n_sn: NUMBER OF SN TO GENERATE #(Optional)
    sn_rate: rate of SN/Mpc^3/year #(Optional, default=3e-5)
    rate_pw: rate = sn_rate*(1+z)^rate_pw #(Optional, default=0)
    duration: DURATION OF THE SURVEY #(Optional, default given by cadence file)
    nep_cut: [[nep_min1,Tmin,Tmax],[nep_min2,Tmin2,Tmax2,'filter1'],...] EP CUTS #(Optional defaut >= 1 ep)
    randseed: RANDSEED TO REPRODUCE SIMULATION #(Optional default random)
    z_range: [ZMIN,ZMAX]
    v_cmb: OUR PECULIAR VELOCITY #(Optional, default = 369.82 km/s)
    M0: SN ABSOLUT MAGNITUDE
    mag_smear: SN INTRINSIC SMEARING
    smear_mod: 'G10','C11_i' USE WAVELENGHT DEP MODEL FOR SN INT SCATTERING
cosmology:
    Om: MATTER DENSITY  
    H0: HUBBLE CONSTANT
salt_gen:
    version: 2 or 3
    salt_dir: '/PATH/TO/SALT/MODEL'  
    alpha: STRETCH CORRECTION = alpha*x1
    beta: COLOR CORRECTION = -beta*c   
    mean_x1: MEAN X1 VALUE
    mean_c: MEAN C VALUE
    sig_x1: SIGMA X1   
    sig_c: SIGMA C
 vpec_gen:
     host_file: '/PATH/TO/HOSTFILE'
     mean_vpec: MEAN SN PECULIAR VEL
     sig_vpec: SIGMA VPEC
```

* If the name of bands in the db file doesn't match sncosmo bands you can use the key band_dic to translate filters names
* If you don't set the filter name item in nep_cut, the cut apply to all the band
* For wavelength dependent model, nomanclature follow arXiv:1209.2482 -> 'G10' for Guy et al. 2010 model, 'C11' or 'C11_0' for Chotard et al. model with correlation between U' and U = 0, 'C11_1' for Cor(U',U) = 1 and 'C11_2' for Cor(U',U) = -1

## Observation DataBase file:
It's a sql database file which contain cadence information. It's used to find obs epoch and their noise.

The required data keys are resumed in the next table

| expMJD | filter | fieldRA (rad) |  fieldDec (rad) | fiveSigmaDepth |
| :-----------: | :-----: | :----------: | :----------: | :--------------------: |
| Obs time| Obs band | Right ascension of the obs field| Declinaison of the obs field   |  Limiting magnitude at 5 sigma |

## Host file
The host file contain coordinates and peculiar velocities to simulate SN, the needed keys are given in the next table

| redshift | ra (rad) | dec (rad) | vp_sight (km/s) |
| :-----------: | :-----: | :----------: | :----------: |
| Redshift of the host | Right ascension of the host | Declinaison of the host | Velocity along the line of sight |

## Usage and output
```
from snsim import sn_sim

sim = sn_sim('yaml_cfg_file.yml')
sim.simulate()
```

The result is stored in sim.sim_lc table which each entry is a SN light curve. Metadata are given by
```
sim.sim_lc[i].meta
```
The list of ligth curves metadata is given in the following table

| z | sim_t0 | sim_x0 | sim_x1 | sim_c | vpec (km/s) | zcos | zpec | z2cmb | zCMB | ra (rad) | dec (rad) |  sn id  | mb | mu | m_smear |
| :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: | :------------: |
|  Observed redshift | Peaktime | SALT2 x0 (normalisation) parameter  | SALT2 x1 (stretch) parameter  | SALT2 c (color) parameter | Peculiar velocity  | Cosmological redshift  | Peculiar velocity redshift | CMB motion redshift | CMB frame redshift | SN right ascension   |  SN declinaison |  SN identification number | SN magnitude in restframe Bessell B | Simulated distance modulli | Coherent smear term |

## Script launch
The program can be launch with the ./sripts/launch_sim.py python script.

The script use argparse to change parameters:
```
>python3 launch_sim.py '/PATH/TO/YAMLFILE' -fit (optional if you want to fit) --any_config_key value (overwrite yaml configuration or add param)
```
If the config keys is a float or an int just type as :
```
>python3 launch_sim.py '/PATH/TO/YAMLFILE' --int_or_float_key value_nbr
```
If the config key is a dict you have to pass it like a yaml string :
```
>python3 launch_sim.py '/PATH/TO/YAMLFILE' --dic_key "{'key1': value1, 'key2': value2, ...}"
```
If the config keys is a list you have to pass it by separate item by space :
```
>python3 launch_sim.py '/PATH/TO/YAMLFILE' --list_key item1 item2 item3
```
In the case of nep_cut key you can pass an int or pass list by typing --nep_cut multiple times, note that filter argument is optional:
```
#nep_cut is just an int
>python3 launch_sim.py '/PATH/TO/YAMLFILE' --nep_cut minimal_nbr_of_epoch

#Multiple cuts
>python3 launch_sim.py '/PATH/TO/YAMLFILE' --nep_cut ep_nbr1 time_inf1 time_sup1 optional_filter1 --nep_cut ep_nbr2 time_inf2 time_sup2 optional_filter2
```
## Fit and open_sim class
You can direct fit after running the simulation 
```
# Fit 1 lc by id
sim.fit_lc(id)

# Fit all the lcs
sim.fit_lc()

# Write the fit
sim.write_fit()
```
Or you can open register open sim file .fits or .pkl with the open_sim class
```
from snsim import open_sim

sim = open_sim('sim_file.pkl/.fits',SALT2_dir)
```

