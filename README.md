import { useState, useEffect } from 'react';
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Calendar, Hotel, User, Mail, Phone, Settings, Plus, Search, MapPin } from 'lucide-react';

export default function RoomaKuApp() {
  const [checkIn, setCheckIn] = useState("");
  const [checkOut, setCheckOut] = useState("");
  const [rooms, setRooms] = useState([
    {
      name: "Deluxe Room",
      description: "Kamar mewah dengan pemandangan kota dan fasilitas lengkap",
      price: "500000",
      image: "data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMzAwIiBoZWlnaHQ9IjIwMCIgdmlld0JveD0iMCAwIDMwMCAyMDAiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+CjxyZWN0IHdpZHRoPSIzMDAiIGhlaWdodD0iMjAwIiBmaWxsPSIjZjNmNGY2Ii8+CjxyZWN0IHg9IjUwIiB5PSI1MCIgd2lkdGg9IjIwMCIgaGVpZ2h0PSIxMDAiIGZpbGw9IiNlNWU3ZWIiIHJ4PSI4Ii8+Cjx0ZXh0IHg9IjE1MCIgeT0iMTA1IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBmaWxsPSIjNjc3NDhkIiBmb250LXNpemU9IjE0Ij5EZWx1eGUgUm9vbTwvdGV4dD4KPC9zdmc+",
      available: true
    },
    {
      name: "Standard Room",
      description: "Kamar nyaman dengan fasilitas standar untuk perjalanan bisnis",
      price: "300000",
      image: "data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMzAwIiBoZWlnaHQ9IjIwMCIgdmlld0JveD0iMCAwIDMwMCAyMDAiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+CjxyZWN0IHdpZHRoPSIzMDAiIGhlaWdodD0iMjAwIiBmaWxsPSIjZjNmNGY2Ii8+CjxyZWN0IHg9IjUwIiB5PSI1MCIgd2lkdGg9IjIwMCIgaGVpZ2h0PSIxMDAiIGZpbGw9IiNkMWZhZTUiIHJ4PSI4Ii8+Cjx0ZXh0IHg9IjE1MCIgeT0iMTA1IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiBmaWxsPSIjMzc0MTUxIiBmb250LXNpemU9IjE0Ij5TdGFuZGFyZCBSb29tPC90ZXh0Pgo8L3N2Zz4=",
      available: true
    }
  ]);
  const [filteredRooms, setFilteredRooms] = useState([]);
  const [isAdmin, setIsAdmin] = useState(false);
  const [adminInputMode, setAdminInputMode] = useState(false);
  const [adminPassword, setAdminPassword] = useState("");
  const [newRoom, setNewRoom] = useState({
    name: "",
    description: "",
    price: "",
    image: "",
    available: true,
  });
  const [rekening, setRekening] = useState("BCA: 1234567890 - Hotel RoomaKu");
  const [showToast, setShowToast] = useState(false);
  const [toastMessage, setToastMessage] = useState("");
  const [roomAvailability, setRoomAvailability] = useState({
    "Deluxe Room_2025-07-25": 3,
    "Deluxe Room_2025-07-26": 2,
    "Deluxe Room_2025-07-27": 1,
    "Standard Room_2025-07-25": 5,
    "Standard Room_2025-07-26": 4,
    "Standard Room_2025-07-27": 3,
  });
  const [selectedDate, setSelectedDate] = useState("");
  const [selectedRoomName, setSelectedRoomName] = useState("");
  const [availableCount, setAvailableCount] = useState(0);
  const [bookingRoom, setBookingRoom] = useState(null);
  const [userInfo, setUserInfo] = useState({ name: "", email: "", whatsapp: "" });
  const [bookings, setBookings] = useState([
    {
      room: "Deluxe Room",
      checkIn: "2025-07-25",
      checkOut: "2025-07-27",
      name: "John Doe",
      email: "john@example.com",
      whatsapp: "081234567890"
    }
  ]);
  const [editingRoom, setEditingRoom] = useState(null);
  const [editRoomData, setEditRoomData] = useState({
    name: "",
    description: "",
    price: "",
    image: "",
    available: true,
  });

  const searchRooms = () => {
    if (!checkIn || !checkOut) {
      showToastMessage("Silakan pilih tanggal check-in dan check-out");
      return;
    }

    const checkInDate = new Date(checkIn);
    const checkOutDate = new Date(checkOut);
    
    if (checkInDate >= checkOutDate) {
      showToastMessage("Tanggal check-out harus setelah check-in");
      return;
    }

    const result = rooms.map((room) => {
      let minAvailable = Infinity;
      for (let d = new Date(checkInDate); d < checkOutDate; d.setDate(d.getDate() + 1)) {
        const key = `${room.name}_${d.toISOString().split("T")[0]}`;
        const availability = parseInt(roomAvailability[key] || 0);
        if (availability < minAvailable) {
          minAvailable = availability;
        }
      }
      return { ...room, availableCount: minAvailable === Infinity ? 0 : minAvailable };
    });
    
    setFilteredRooms(result);
    showToastMessage(`Ditemukan ${result.length} tipe kamar untuk periode tersebut`);
  };

  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file && file.size > 1024 * 1024) {
      showToastMessage("Ukuran file terlalu besar (maksimal 1MB)");
      return;
    }
    const reader = new FileReader();
    reader.onloadend = () => {
      setNewRoom({ ...newRoom, image: reader.result });
    };
    if (file) reader.readAsDataURL(file);
  };

  const addRoom = () => {
    if (!newRoom.name || !newRoom.description || !newRoom.price) {
      showToastMessage("Semua field harus diisi");
      return;
    }
    
    setRooms([...rooms, { ...newRoom, id: Date.now() }]);
    setNewRoom({ name: "", description: "", price: "", image: "", available: true });
    showToastMessage("Kamar berhasil ditambahkan");
  };

  const deleteRoom = (index) => {
    const updatedRooms = rooms.filter((_, i) => i !== index);
    setRooms(updatedRooms);
    showToastMessage("Kamar berhasil dihapus");
  };

  const startEditRoom = (room, index) => {
    setEditingRoom(index);
    setEditRoomData({
      name: room.name,
      description: room.description,
      price: room.price,
      image: room.image,
      available: room.available,
    });
  };

  const cancelEditRoom = () => {
    setEditingRoom(null);
    setEditRoomData({
      name: "",
      description: "",
      price: "",
      image: "",
      available: true,
    });
  };

  const saveEditRoom = () => {
    if (!editRoomData.name || !editRoomData.description || !editRoomData.price) {
      showToastMessage("Semua field harus diisi");
      return;
    }

    const updatedRooms = rooms.map((room, index) => 
      index === editingRoom ? { ...editRoomData, id: room.id || Date.now() } : room
    );
    
    setRooms(updatedRooms);
    setEditingRoom(null);
    setEditRoomData({
      name: "",
      description: "",
      price: "",
      image: "",
      available: true,
    });
    showToastMessage("Kamar berhasil diperbarui");
  };

  const handleEditImageUpload = (e) => {
    const file = e.target.files[0];
    if (file && file.size > 1024 * 1024) {
      showToastMessage("Ukuran file terlalu besar (maksimal 1MB)");
      return;
    }
    const reader = new FileReader();
    reader.onloadend = () => {
      setEditRoomData({ ...editRoomData, image: reader.result });
    };
    if (file) reader.readAsDataURL(file);
  };

  const handleBooking = (room) => {
    if (!checkIn || !checkOut) {
      showToastMessage("Silakan pilih tanggal check-in dan check-out terlebih dahulu");
      return;
    }
    setBookingRoom(room);
  };

  const confirmBooking = () => {
    if (!userInfo.name || !userInfo.email || !userInfo.whatsapp) {
      showToastMessage("Semua informasi kontak harus diisi");
      return;
    }

    const newBooking = {
      id: Date.now(),
      room: bookingRoom.name,
      checkIn,
      checkOut,
      name: userInfo.name,
      email: userInfo.email,
      whatsapp: userInfo.whatsapp,
      status: "confirmed",
      bookingDate: new Date().toISOString().split('T')[0]
    };
    
    setBookings([...bookings, newBooking]);
    showToastMessage("Pemesanan berhasil dikonfirmasi!");
    setBookingRoom(null);
    setUserInfo({ name: "", email: "", whatsapp: "" });
  };

  const cancelBooking = (bookingId) => {
    const updatedBookings = bookings.filter(b => b.id !== bookingId);
    setBookings(updatedBookings);
    showToastMessage("Pemesanan berhasil dibatalkan");
  };

  const showToastMessage = (message) => {
    setToastMessage(message);
    setShowToast(true);
    setTimeout(() => setShowToast(false), 3000);
  };

  const handleAdminLogin = () => {
    if (adminPassword === "admin123") {
      setIsAdmin(true);
      setAdminInputMode(false);
      setAdminPassword("");
      showToastMessage("Login admin berhasil");
    } else {
      showToastMessage("Password salah");
    }
  };

  const handleAvailabilityChange = () => {
    if (!selectedDate || !selectedRoomName) {
      showToastMessage("Pilih tanggal dan nama kamar");
      return;
    }
    const key = `${selectedRoomName}_${selectedDate}`;
    setRoomAvailability({
      ...roomAvailability,
      [key]: availableCount,
    });
    showToastMessage("Ketersediaan kamar berhasil diperbarui");
  };

  const formatPrice = (price) => {
    return new Intl.NumberFormat('id-ID', {
      style: 'currency',
      currency: 'IDR',
      minimumFractionDigits: 0
    }).format(price);
  };

  const getTodayDate = () => {
    return new Date().toISOString().split('T')[0];
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
      {showToast && (
        <div className="fixed top-4 right-4 bg-green-500 text-white px-6 py-3 rounded-lg shadow-lg z-50 animate-pulse">
          ✅ {toastMessage}
        </div>
      )}

      {/* Header */}
      <div className="text-center mb-8">
        <h1 className="text-4xl font-bold text-gray-800 mb-2 flex items-center justify-center gap-3">
          <Hotel className="text-blue-600" size={40} />
          RoomaKu Hotel
        </h1>
        <p className="text-gray-600">Sistem Pemesanan Kamar Hotel Modern</p>
      </div>

      {/* Admin Login Button */}
      <div className="flex justify-end mb-6">
        {!isAdmin && !adminInputMode && (
          <Button 
            onClick={() => setAdminInputMode(true)}
            variant="outline"
            className="flex items-center gap-2"
          >
            <Settings size={16} />
            Admin Login
          </Button>
        )}
        
        {adminInputMode && (
          <div className="flex gap-2">
            <Input
              type="password"
              placeholder="Password Admin"
              value={adminPassword}
              onChange={(e) => setAdminPassword(e.target.value)}
              className="w-40"
            />
            <Button onClick={handleAdminLogin}>Login</Button>
            <Button 
              variant="outline" 
              onClick={() => {
                setAdminInputMode(false);
                setAdminPassword("");
              }}
            >
              Batal
            </Button>
          </div>
        )}
        
        {isAdmin && (
          <Button 
            onClick={() => setIsAdmin(false)}
            variant="outline"
            className="text-red-600"
          >
            Logout Admin
          </Button>
        )}
      </div>

      {/* Search Section */}
      <Card className="max-w-4xl mx-auto mb-8 shadow-lg">
        <CardContent className="p-6">
          <h2 className="text-2xl font-semibold mb-4 flex items-center gap-2">
            <Search className="text-blue-600" size={24} />
            Cari Kamar Tersedia
          </h2>
          <div className="grid md:grid-cols-3 gap-4">
            <div>
              <label className="block text-sm font-medium mb-2">Check-in</label>
              <Input
                type="date"
                value={checkIn}
                onChange={(e) => setCheckIn(e.target.value)}
                min={getTodayDate()}
                className="w-full"
              />
            </div>
            <div>
              <label className="block text-sm font-medium mb-2">Check-out</label>
              <Input
                type="date"
                value={checkOut}
                onChange={(e) => setCheckOut(e.target.value)}
                min={checkIn || getTodayDate()}
                className="w-full"
              />
            </div>
            <div className="flex items-end">
              <Button 
                onClick={searchRooms}
                className="w-full bg-blue-600 hover:bg-blue-700"
              >
                <Search size={16} className="mr-2" />
                Cari Kamar
              </Button>
            </div>
          </div>
        </CardContent>
      </Card>

      {/* Available Rooms */}
      {filteredRooms.length > 0 && (
        <Card className="max-w-4xl mx-auto mb-8 shadow-lg">
          <CardContent className="p-6">
            <h2 className="text-2xl font-semibold mb-4">Kamar Tersedia</h2>
            <div className="grid md:grid-cols-2 gap-6">
              {filteredRooms.map((room, index) => (
                <div key={index} className="border rounded-lg p-4 bg-white shadow-sm">
                  {room.image && (
                    <img
                      src={room.image}
                      alt={room.name}
                      className="w-full h-48 object-cover rounded-lg mb-4"
                    />
                  )}
                  <h3 className="text-xl font-semibold mb-2">{room.name}</h3>
                  <p className="text-gray-600 mb-3">{room.description}</p>
                  <div className="flex justify-between items-center mb-3">
                    <span className="text-2xl font-bold text-blue-600">
                      {formatPrice(room.price)}/malam
                    </span>
                    <span className="text-sm text-gray-500">
                      Tersedia: {room.availableCount} kamar
                    </span>
                  </div>
                  <Button 
                    onClick={() => handleBooking(room)}
                    disabled={room.availableCount === 0}
                    className="w-full"
                  >
                    {room.availableCount === 0 ? 'Tidak Tersedia' : 'Pesan Sekarang'}
                  </Button>
                </div>
              ))}
            </div>
          </CardContent>
        </Card>
      )}

      {/* All Rooms Display (when no search) */}
      {filteredRooms.length === 0 && (
        <Card className="max-w-4xl mx-auto mb-8 shadow-lg">
          <CardContent className="p-6">
            <h2 className="text-2xl font-semibold mb-4">Semua Tipe Kamar</h2>
            <div className="grid md:grid-cols-2 gap-6">
              {rooms.map((room, index) => (
                <div key={index} className="border rounded-lg p-4 bg-white shadow-sm">
                  {room.image && (
                    <img
                      src={room.image}
                      alt={room.name}
                      className="w-full h-48 object-cover rounded-lg mb-4"
                    />
                  )}
                  <h3 className="text-xl font-semibold mb-2">{room.name}</h3>
                  <p className="text-gray-600 mb-3">{room.description}</p>
                  <div className="flex justify-between items-center mb-3">
                    <span className="text-2xl font-bold text-blue-600">
                      {formatPrice(room.price)}/malam
                    </span>
                  </div>
                  <p className="text-sm text-gray-500 mb-3">
                    Pilih tanggal untuk melihat ketersediaan
                  </p>
                </div>
              ))}
            </div>
          </CardContent>
        </Card>
      )}

      {/* Booking Modal */}
      {bookingRoom && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
          <Card className="max-w-md w-full">
            <CardContent className="p-6">
              <h3 className="text-xl font-semibold mb-4">Konfirmasi Pemesanan</h3>
              <div className="mb-4">
                <p><strong>Kamar:</strong> {bookingRoom.name}</p>
                <p><strong>Harga:</strong> {formatPrice(bookingRoom.price)}/malam</p>
                <p><strong>Check-in:</strong> {checkIn}</p>
                <p><strong>Check-out:</strong> {checkOut}</p>
                <p><strong>Tersedia:</strong> {bookingRoom.availableCount} kamar</p>
              </div>
              
              <div className="space-y-3 mb-4">
                <Input
                  placeholder="Nama Lengkap"
                  value={userInfo.name}
                  onChange={(e) => setUserInfo({...userInfo, name: e.target.value})}
                />
                <Input
                  type="email"
                  placeholder="Email"
                  value={userInfo.email}
                  onChange={(e) => setUserInfo({...userInfo, email: e.target.value})}
                />
                <Input
                  placeholder="Nomor WhatsApp"
                  value={userInfo.whatsapp}
                  onChange={(e) => setUserInfo({...userInfo, whatsapp: e.target.value})}
                />
              </div>

              <div className="mb-4 p-3 bg-gray-50 rounded">
                <p className="text-sm font-medium">Informasi Rekening:</p>
                <p className="text-sm text-gray-600">{rekening}</p>
              </div>

              <div className="flex gap-2">
                <Button onClick={confirmBooking} className="flex-1">
                  Konfirmasi Pesanan
                </Button>
                <Button 
                  variant="outline" 
                  onClick={() => setBookingRoom(null)}
                  className="flex-1"
                >
                  Batal
                </Button>
              </div>
            </CardContent>
          </Card>
        </div>
      )}

      {/* Admin Panel */}
      {isAdmin && (
        <Card className="max-w-4xl mx-auto mb-8 shadow-lg">
          <CardContent className="p-6 bg-gray-50">
            <h2 className="text-2xl font-bold mb-6 text-center">Admin Panel</h2>

            {/* Add New Room */}
            <div className="mb-8 p-4 border rounded-lg bg-white">
              <h3 className="text-lg font-semibold mb-4 flex items-center gap-2">
                <Plus size={20} />
                Tambah Kamar Baru
              </h3>
              <div className="grid md:grid-cols-2 gap-4 mb-4">
                <Input
                  placeholder="Nama Kamar"
                  value={newRoom.name}
                  onChange={(e) => setNewRoom({...newRoom, name: e.target.value})}
                />
                <Input
                  placeholder="Harga per malam"
                  type="number"
                  value={newRoom.price}
                  onChange={(e) => setNewRoom({...newRoom, price: e.target.value})}
                />
              </div>
              <Input
                placeholder="Deskripsi kamar"
                value={newRoom.description}
                onChange={(e) => setNewRoom({...newRoom, description: e.target.value})}
                className="mb-4"
              />
              <Input
                type="file"
                accept="image/*"
                onChange={handleImageUpload}
                className="mb-4"
              />
              {newRoom.image && (
                <img
                  src={newRoom.image}
                  alt="Preview"
                  className="w-full max-w-xs rounded shadow mb-4"
                />
              )}
              <Button onClick={addRoom} className="w-full">
                <Plus size={16} className="mr-2" />
                Tambah Kamar
              </Button>
            </div>

            {/* Room Management */}
            <div className="mb-8 p-4 border rounded-lg bg-white">
              <h3 className="text-lg font-semibold mb-4">Kelola Kamar</h3>
              <div className="space-y-4">
                {rooms.map((room, index) => (
                  <div key={index} className="border rounded-lg p-4">
                    {editingRoom === index ? (
                      // Edit Mode
                      <div className="space-y-4">
                        <div className="grid md:grid-cols-2 gap-4">
                          <Input
                            placeholder="Nama Kamar"
                            value={editRoomData.name}
                            onChange={(e) => setEditRoomData({...editRoomData, name: e.target.value})}
                          />
                          <Input
                            placeholder="Harga per malam"
                            type="number"
                            value={editRoomData.price}
                            onChange={(e) => setEditRoomData({...editRoomData, price: e.target.value})}
                          />
                        </div>
                        <Input
                          placeholder="Deskripsi kamar"
                          value={editRoomData.description}
                          onChange={(e) => setEditRoomData({...editRoomData, description: e.target.value})}
                        />
                        <div>
                          <Input
                            type="file"
                            accept="image/*"
                            onChange={handleEditImageUpload}
                            className="mb-2"
                          />
                          {editRoomData.image && (
                            <img
                              src={editRoomData.image}
                              alt="Preview"
                              className="w-32 h-24 object-cover rounded shadow"
                            />
                          )}
                        </div>
                        <div className="flex items-center gap-4">
                          <label className="flex items-center gap-2">
                            <input
                              type="checkbox"
                              checked={editRoomData.available}
                              onChange={(e) => setEditRoomData({...editRoomData, available: e.target.checked})}
                              className="rounded"
                            />
                            <span className="text-sm">Kamar tersedia</span>
                          </label>
                        </div>
                        <div className="flex gap-2">
                          <Button onClick={saveEditRoom} size="sm" className="bg-green-600 hover:bg-green-700">
                            Simpan
                          </Button>
                          <Button onClick={cancelEditRoom} variant="outline" size="sm">
                            Batal
                          </Button>
                        </div>
                      </div>
                    ) : (
                      // View Mode
                      <div className="flex justify-between items-start">
                        <div className="flex gap-4">
                          {room.image && (
                            <img
                              src={room.image}
                              alt={room.name}
                              className="w-20 h-16 object-cover rounded shadow"
                            />
                          )}
                          <div>
                            <h4 className="font-medium text-lg">{room.name}</h4>
                            <p className="text-sm text-gray-600 mb-1">{room.description}</p>
                            <p className="text-sm font-semibold text-blue-600">{formatPrice(room.price)}/malam</p>
                            <p className="text-xs text-gray-500">
                              Status: {room.available ? 'Aktif' : 'Tidak Aktif'}
                            </p>
                          </div>
                        </div>
                        <div className="flex gap-2">
                          <Button 
                            variant="outline" 
                            size="sm"
                            onClick={() => startEditRoom(room, index)}
                            className="text-blue-600 hover:bg-blue-50"
                          >
                            Edit
                          </Button>
                          <Button 
                            variant="outline" 
                            size="sm"
                            onClick={() => deleteRoom(index)}
                            className="text-red-600 hover:bg-red-50"
                          >
                            Hapus
                          </Button>
                        </div>
                      </div>
                    )}
                  </div>
                ))}
              </div>
            </div>

            {/* Room Availability */}
            <div className="mb-8 p-4 border rounded-lg bg-white">
              <h3 className="text-lg font-semibold mb-4">Atur Ketersediaan Kamar</h3>
              <div className="grid md:grid-cols-4 gap-4 mb-4">
                <select
                  value={selectedRoomName}
                  onChange={(e) => setSelectedRoomName(e.target.value)}
                  className="border rounded px-3 py-2"
                >
                  <option value="">Pilih Kamar</option>
                  {rooms.map((room, index) => (
                    <option key={index} value={room.name}>{room.name}</option>
                  ))}
                </select>
                <Input
                  type="date"
                  value={selectedDate}
                  onChange={(e) => setSelectedDate(e.target.value)}
                  min={getTodayDate()}
                />
                <Input
                  type="number"
                  placeholder="Jumlah tersedia"
                  value={availableCount}
                  onChange={(e) => setAvailableCount(parseInt(e.target.value) || 0)}
                  min="0"
                />
                <Button onClick={handleAvailabilityChange}>
                  Update
                </Button>
              </div>
            </div>

            {/* Payment Info */}
            <div className="mb-8 p-4 border rounded-lg bg-white">
              <h3 className="text-lg font-semibold mb-4">Informasi Rekening</h3>
              <Input
                placeholder="Informasi rekening untuk pembayaran"
                value={rekening}
                onChange={(e) => setRekening(e.target.value)}
                className="mb-4"
              />
            </div>

            {/* Bookings List */}
            <div className="p-4 border rounded-lg bg-white">
              <h3 className="text-lg font-semibold mb-4">Daftar Pemesanan</h3>
              {bookings.length === 0 ? (
                <p className="text-gray-500">Belum ada pemesanan.</p>
              ) : (
                <div className="space-y-4">
                  {bookings.map((booking) => (
                    <div key={booking.id} className="border p-4 rounded-lg">
                      <div className="grid md:grid-cols-2 gap-4">
                        <div>
                          <p><strong>Tamu:</strong> {booking.name}</p>
                          <p><strong>Email:</strong> {booking.email}</p>
                          <p><strong>WhatsApp:</strong> {booking.whatsapp}</p>
                        </div>
                        <div>
                          <p><strong>Kamar:</strong> {booking.room}</p>
                          <p><strong>Check-in:</strong> {booking.checkIn}</p>
                          <p><strong>Check-out:</strong> {booking.checkOut}</p>
                          <p><strong>Status:</strong> {booking.status}</p>
                        </div>
                      </div>
                      <div className="mt-3 flex gap-2">
                        <Button 
                          size="sm" 
                          variant="outline"
                          onClick={() => cancelBooking(booking.id)}
                          className="text-red-600"
                        >
                          Batalkan Pesanan
                        </Button>
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          </CardContent>
        </Card>
      )}

      {/* Footer */}
      <div className="text-center text-gray-600 mt-8">
        <p>&copy; 2025 RoomaKu Hotel. Sistem pemesanan kamar modern dan mudah digunakan.</p>
      </div>
    </div>
  );
}
