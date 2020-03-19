# Bird Flocking Simulation
**Rika Dewi/13517147**

https://youtu.be/KXzHG_e_2O8


Simulasi bird flocking behavior dapat dilakukan dengan menggabungkan 3 buah prinsip:
1. Separation
	Menghindari flockmates yang berkumpul menjadi satu. Gerakan ini sudah diimplementasikan pada `example.cpp` dengan mengaktifkan parameter `repulsion`.
2. Alignment
	Mengarahkan gerakan untuk berjalan ke arah rata-rata kecepatan flockmates terdekat di sekitarnya.
3. Cohesion
	Mengarahkan gerakan ke rata-rata posisi (pusat massa) dari flockmates terdekat di sekitarnya.
	
# Implementasi

Pada implementasinya, saya melakukan update pada `example.cpp`dengan menambahkan 2 buah fungsi yaitu untuk alignment dan cohesion. Selain itu perubahan juga dilakukan di fungsi `idle` untuk mengubah velocity dari setiap partikel.

## Align Function

Pertama-tama kita menghitung rata-rata arah dari partikel lain di sekitar partikel yang ditinjau. 
```
// Get average velocity of local flockmates
for(int i = 0; i < num_particles; i++) {
    float sum = 0;
    for (int j = 0; j < 3; j++) {
        sum += pow(particles[i].position[j] - p.position[j], 2);
    }
    if(pow(sum, 0.5) < perception) {    
        for (int j = 0; j < 3; j++) {
            avg_vec[j] += particles[i].velocity[j];
        }
        total += 1;
    }
}
```
Cara mengukur jarak antara partikel yang tinjau (`p`) terhadap partikel lainnya yaitu dengan menghitung akar kuadrat dari hasil kuadratik penjumlahan selisih posisi terhadap titik yang ditinjau. Di sini `perception` merupakan parameter yang mendefinisikan lokal yaitu sebesar 1 satuan di sekitar partikel yang ditinjau. Jika flockmate terdapat di dalam radius lokal partikel yang ditinjau, maka nilai kecepatannya akan disimpan.

Setelah mendapatkan jumlah dari semua kecepatan lokal, maka akan dihitung vektor arah perubahan kecepatan dari partikel yang ditinjau.
```
// Normalize average velocity with respect to given particle
if (total > 0) {
    for (int j = 0; j < 3; j++) {
        avg_vec[j] /= total;
    }
    float sum = 0;
    for (int j = 0; j < 3; j++) {
        sum += pow(avg_vec[j], 2);
    }
    float norm = pow(sum, 0.5);
    for (int j = 0; j < 3; j++) {
        avg_vec[j] = (avg_vec[j]/norm) * speed;
    }
    for (int j = 0; j < 3; j++) {
        steering[j] = avg_vec[j] - p.velocity[j];
    }

}
```
Nilai dari `avg_vec` akan dibagi dengan besar dari vektor tersebut untuk mendapatkan arahnya saja. Arah ini akan dikalikan dengan kecepatan partikel. Hal ini untuk mendapatkan vektor hasil yang telah dinormalisasi sehingga tidak terjadi perubahan besar kecepatan partikel. Hasil dari vektor ini kemudian akan dikurangkan dengan kecepatan partikel yang ditinjau untuk mendapatkan vektor perubahannya.


Berikut kodenya secara lengkap.
```
vec4 align(particle p) {
    // Initialize
    vec4 steering;
    steering[0] = 0;
    steering[1] = 0;
    steering[2] = 0;
    steering[3] = 0;
    int total = 0;
    vec4 avg_vec;
    avg_vec[0] = 0;
    avg_vec[1] = 0;
    avg_vec[2] = 0;
    avg_vec[3] = 0;

    // Get average velocity of local flockmates
    for(int i = 0; i < num_particles; i++) {
        float sum = 0;
        for (int j = 0; j < 3; j++) {
            sum += pow(particles[i].position[j] - p.position[j], 2);
        }
        if(pow(sum, 0.5) < perception) {    
            for (int j = 0; j < 3; j++) {
                avg_vec[j] += particles[i].velocity[j];
            }
            total += 1;
        }
    }

    // Normalize average velocity with respect to given particle
    if (total > 0) {
        for (int j = 0; j < 3; j++) {
            avg_vec[j] /= total;
        }
        float sum = 0;
        for (int j = 0; j < 3; j++) {
            sum += pow(avg_vec[j], 2);
        }
        float norm = pow(sum, 0.5);
        for (int j = 0; j < 3; j++) {
            avg_vec[j] = (avg_vec[j]/norm) * speed;
        }
        for (int j = 0; j < 3; j++) {
            steering[j] = avg_vec[j] - p.velocity[j];
        }

    }

    return steering;
}
```

## Cohesion Function
Untuk menentukan cohesion dibutuhkan nilai pusat massa (center of mass) dari partikel yang berada di sekitar partikel yang ditinjau. Center of mass ini dapat dihitung dengan mengkalikan jarak titik pusat massa dengan massa dari partikel tersebut. Pada kesempatan kali ini, semua partikel pada `example.cpp` sudah diinisilisasi dengan massa yang homogen yaitu sebesar 1. Karena itu variabel massa untuk menghitung center of mass dapat diabaikan. 

```
// Get sum mass of local flockmates
for(int i = 0; i < num_particles; i++) {
    float sum = 0;
    for (int j = 0; j < 3; j++) {
        sum += pow(particles[i].position[j] - p.position[j], 2);
    }
    if(pow(sum, 0.5) < perception) {    
        for (int j = 0; j < 3; j++) {
            center_of_mass[j] += particles[i].position[j];
        }
        total += 1;
    }
}
```
Dapat dilihat bahwa seperti pada fungsi `align`, pertama-tama dihitung jarak dari flockmates terhadap partikel yang ditinjau. Jika jaraknya lebih kecil dari `perception` (parameter user) maka nilai posisi partikel tersebut akan disimpan.

```
// Normalize center of mass to get vector for cohesien
if (total > 0) {
    for (int j = 0; j < 3; j++) {
        center_of_mass[j] /= total;
    }

    for (int j = 0; j < 3; j++) {
        center_of_mass[j] -= p.position[j];
    }

    float sum = 0;
    for (int j = 0; j < 3; j++) {
        sum += pow(center_of_mass[j], 2);
    }
    float norm = pow(sum, 0.5);
    if (norm > 0) {
        for (int j = 0; j < 3; j++) {
            center_of_mass[j] = (center_of_mass[j]/norm) * speed;
        }
    }
    for (int j = 0; j < 3; j++) {
        steering[j] = center_of_mass[j] - p.velocity[j];
    }

    sum = 0;
    for (int j = 0; j < 3; j++) {
        sum += pow(steering[j], 2);
    }
    norm = pow(sum, 0.5);
    if (norm > speed) {
        for (int j = 0; j < 3; j++) {
            steering[j] = (steering[j]/norm) * speed;
        }
    }

}
```
Setelah didapat titik center of mass dari sekumpulan partikel di sekitar partikel tinjauan, maka akan dicari vektor yang mengarahkan partikel tinjaun ke pusat massa. Vektor ini didapat dari pengurangan 2 titik (center of mass - posisi partikel). Nilai ini kemudian dinormalisasi seperti pada fungsi `align`. Hasil dari vektor arah ini kemudian dikurangi ke kecepatan partikel tinjauan untuk mendapatkan perubahan vektor kecepatan titik tinjauan. Hasilnya pun dinormalisasi lagi.

Berikut kodenya secara lengkap
```
vec4 cohesion(particle p) {
    vec4 steering;
    steering[0] = 0;
    steering[1] = 0;
    steering[2] = 0;
    steering[3] = 0;
    int total = 0;
    vec4 center_of_mass;
    center_of_mass[0] = 0;
    center_of_mass[1] = 0;
    center_of_mass[2] = 0;
    center_of_mass[3] = 0;

    // Get sum mass of local flockmates
    for(int i = 0; i < num_particles; i++) {
        float sum = 0;
        for (int j = 0; j < 3; j++) {
            sum += pow(particles[i].position[j] - p.position[j], 2);
        }
        if(pow(sum, 0.5) < perception) {    
            for (int j = 0; j < 3; j++) {
                center_of_mass[j] += particles[i].position[j];
            }
            total += 1;
        }
    }

    // Normalize center of mass to get vector for cohesien
    if (total > 0) {
        for (int j = 0; j < 3; j++) {
            center_of_mass[j] /= total;
        }

        for (int j = 0; j < 3; j++) {
            center_of_mass[j] -= p.position[j];
        }

        float sum = 0;
        for (int j = 0; j < 3; j++) {
            sum += pow(center_of_mass[j], 2);
        }
        float norm = pow(sum, 0.5);
        if (norm > 0) {
            for (int j = 0; j < 3; j++) {
                center_of_mass[j] = (center_of_mass[j]/norm) * speed;
            }
        }
        for (int j = 0; j < 3; j++) {
            steering[j] = center_of_mass[j] - p.velocity[j];
        }

        sum = 0;
        for (int j = 0; j < 3; j++) {
            sum += pow(steering[j], 2);
        }
        norm = pow(sum, 0.5);
        if (norm > speed) {
            for (int j = 0; j < 3; j++) {
                steering[j] = (steering[j]/norm) * speed;
            }
        }

    }

    return steering;
}
```

## Idle Function
Terakhir, kedua fungsi ini akan dipanggil pada idle function. Selain itu parameter repulsion juga dijadikan `true`.

```
void idle( void ) { 
    int i, j, k; 
    float dt; 
    present_time = glutGet( GLUT_ELAPSED_TIME );
    dt = 0.001 * ( present_time - last_time );
    for ( int i = 0; i < num_particles; i++ ) {
        vec4 align_vec = align(particles[i]);
        vec4 cohesion_vec = cohesion(particles[i]);
        for ( int j = 0; j < 3; j++ ) {
            particles[i].position[j] += dt * particles[i].velocity[j];
            particles[i].velocity[j] += dt * align_vec[j];
            particles[i].velocity[j] += dt * cohesion_vec[j] * cohesion_coef;
            particles[i].velocity[j] += 
                dt * forces( i, j ) / particles[i].mass;
        }
        collision( i );
    }
    if ( repulsion )
        for ( int i = 0; i < num_particles; i++ )
            for ( int k = 0; k < i; k++ ) {
                d2[i][k] = 0.0;
                for ( int j = 0; j < 3; j++ )
                    d2[i][k] += ( particles[i].position[j] -
                                  particles[k].position[j] ) *
                        ( particles[i].position[j] -
                          particles[k].position[j] );
                d2[k][i] = d2[i][k];
            }
    last_time = present_time;
    glutPostRedisplay();
}
```

Kode perubahan secara lengkap dari `example.cpp` dapat dilihat pada `https://github.com/Rikadewi/bird-flocking.git`.