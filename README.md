# Save this file as micropython_gps.py
import time

class MicropythonGPS:
    def __init__(self, local_offset=0):
        self.latitude = [0, 'N']
        self.longitude = [0, 'E']
        self.altitude = 0.0
        self.satellites_in_use = 0
        self.valid = False

    def update(self, data):
        if not data:
            return None
        
        # We look for GPRMC or GPGGA sentences from the satellite
        if data.startswith('$GPRMC') or data.startswith('$GNRMC'):
            parts = data.split(',')
            if len(parts) > 2 and parts[2] == 'A':
                self.valid = True
                # Parse Latitude
                lat = parts[3]
                if lat:
                    self.latitude[0] = float(lat[:2]) + float(lat[2:]) / 60.0
                    self.latitude[1] = parts[4]
                # Parse Longitude
                lon = parts[5]
                if lon:
                    self.longitude[0] = float(lon[:3]) + float(lon[3:]) / 60.0
                    self.longitude[1] = parts[6]
            else:
                self.valid = False
                
        elif data.startswith('$GPGGA') or data.startswith('$GNGGA'):
            parts = data.split(',')
            if len(parts) > 7:
                try:
                    self.satellites_in_use = int(parts[7])
                    self.altitude = float(parts[9])
                except ValueError:
                    pass
        return self.valid
