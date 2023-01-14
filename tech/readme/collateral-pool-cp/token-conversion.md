# Token Conversion

Collateral Pool allows anyone to

* lock C to mint D(s)
* burn D(s) to unlock C
* burn D(s) to mint D(s)

Logic contracts define the conversion rate between each token, but the value of $$R_{BC}$$ must not be reduced:

$$R^1_{BC} ≥ R^0_{BC}$$

Practically, $$R_{BC}$$ should be increased after each conversion due to the Conversion Fee left in the pool.

<figure><img src="https://lh3.googleusercontent.com/AGzpjethMdQy4Dsmoj1JWNDvXY_aI3U3TAZB8RpF9FvWw_H8heD9fEFPyABYH6UD7w-CPQUx4qsLG19qA_oIBatL5UBqYOZgXQEJu8sbeXigQtxALGeNBkfNl6wx70D2OETbEpwmk_x2aoyMi3x8P3bFNqEASNvH8O3fZSaQKcUkiS0DZDvJNB-dWK8EvA" alt=""><figcaption><p>Token Conversion</p></figcaption></figure>
