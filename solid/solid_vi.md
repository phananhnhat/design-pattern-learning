# Các Nguyên Tắc SOLID trong Thiết Kế Hướng Đối Tượng

## Bài Giảng của Robert Martin (Uncle Bob)

Chào buổi tối. Tên tôi là Kyle Jensen. Tôi là giảng viên tại Trường Quản lý Yale, đồng thời là giám đốc các chương trình khởi nghiệp ở đây. Khách mời của chúng ta tối nay là Robert Martin, người sẽ nói về các nguyên tắc SOLID trong thiết kế hướng đối tượng và agile. Chúng ta rất may mắn khi có ông ấy ở đây.

Ông ấy đến vào thời điểm tuyệt vời cho Yale. Như các bạn có thể biết, trường đại học đang đầu tư rất nhiều vào khởi nghiệp. Điều này đang tạo ra, hoặc kết hợp với, chuyên môn truyền thống của trường trong khoa học máy tính và lý thuyết, tạo ra rất nhiều thứ tuyệt vời trên khuôn viên.

Ví dụ, hai năm trước, Yale đã khởi động tech bootcamp. Hack Yale bắt đầu cách đây một năm. Yale Hack đã tập hợp một nghìn lập trình viên từ khắp nơi trên đất nước cho một hackathon kéo dài hai ngày. Học kỳ này, Daniel Boddy đã thay đổi khóa học lập trình nhập môn ở đây, là khóa học lập trình cốt lõi cho các sinh viên không chuyên, thành phát triển ứng dụng di động.

Daniel và tôi sẽ cùng giảng dạy vào mùa xuân năm sau. Khoa học máy tính 113, về khởi nghiệp và lập trình. Chúng tôi có một khóa học về quản lý phát triển phần mềm ở đây tại Trường Quản lý Yale.

Điều này cho thấy có một sự quan tâm lớn tại Yale đối với phát triển phần mềm, và nó vượt qua một vài ngành học khác nhau. Nó có ở Trường Quản lý và Khoa Khoa học Máy tính. Nó có ở Trường Kỹ thuật và rất nhiều nơi khác. Trải dài qua các câu lạc bộ, các tổ chức như Viện Khởi nghiệp Yale, Hack Yale, Câu lạc bộ Khởi nghiệp ở ISTY Hack, tất cả những loại tổ chức này.

Không chỉ có Yale. Có một vài logo ở đó không thuộc về Yale nhưng cho thấy hoạt động trong lĩnh vực này ở cộng đồng địa phương. Chúng ta phải cảm ơn Continuity Control vì đồ ăn tối nay. Đó là một startup địa phương và Ruby shop chứa đầy những hacker và lập trình viên tuyệt vời rất tích cực trong New Haven IO, là một nhóm lập trình viên địa phương mà tôi đã vinh dự được tham gia.

Tôi đã làm việc với Dan và Joel và nhiều người khác đang ở đây để tổ chức nhiều sự kiện cho New Haven IO. Tôi rất vui khi nói rằng đa số mọi người ở đây tối nay thực ra có lẽ không đến từ Yale, điều mà tôi nghĩ là thực sự quan trọng. Theo truyền thống, Yale có vấn đề về "thị trấn và áo choàng", nơi không có nhiều sự tương tác. Tôi nghĩ đó là một vấn đề thực sự.

Đây là một lĩnh vực mà mọi người đều có chung sự quan tâm và sự phấn khích. Thật tuyệt vời khi có những sự kiện như thế này. Chúng tôi rất vui khi có Bob ở đây để nói chuyện với chúng ta. Tôi sẽ mời Dan lên và tự giới thiệu. Dan, tôi sẽ giới thiệu bạn. Dan là một lập trình viên tại Continuity Control. Anh ấy đã là người tổ chức của nhóm Ruby địa phương trong nhiều năm và là người tổ chức tại New Haven IO. Tôi nghĩ anh ấy ở trong ban quản trị của New Haven IO. Về cơ bản anh ấy là một lập trình viên địa phương rất tích cực, rất hỗ trợ cộng đồng địa phương. Tôi nghĩ điều đó thật tuyệt vời. Đó là cách tình bạn của chúng tôi phát triển trong bối cảnh đó. Vậy nên Dan, cảm ơn bạn.

Được rồi. Tôi nên chiếu lên thôi. Xin lỗi. Được rồi, Dan Bernie, Continuity, New Haven IO, phiên bản tóm tắt. Được rồi. Như tôi đã nói, chúng tôi đang tuyển dụng, và tất cả chúng tôi đều có những chiếc ghim ve áo Continuity nhỏ. Chúng tôi làm một số thứ khá thú vị. Chúng tôi hướng tới việc có code thực sự sạch. Chúng tôi có một số dự án thực sự thú vị sắp tới với machine learning, những thứ thú vị như thế. Vì vậy chúng tôi đang tuyển dụng. Đến nói chuyện với chúng tôi.

## Giới Thiệu về Các Thách Thức trong Kỹ Nghệ Phần Mềm

Phần mềm đang tạo ra phần mềm. Đó là một ngành nghề kỳ lạ. Đó là một ngành nghề lộn xộn. Những người trong các bạn là sinh viên có thể chưa khám phá ra điều này vì khi nó chỉ để vui thì nó rất vui. Nhưng khi nó quan trọng, khi bạn có người trả tiền cho phần mềm mà bạn đang viết, hoặc người sử dụng phần mềm của bạn, hoặc trời đất, những người có sinh mạng phụ thuộc vào phần mềm của bạn, thì đó là một ngành nghề kỳ lạ. Đôi khi bạn có thể cảm thấy như chúng ta là một đám trẻ con chạy xung quanh với dây giày chưa buộc, chỉ hy vọng mình không ngã.

May mắn thay, chúng ta có một người chú quan tâm đến chúng ta, đá vào mông chúng ta thỉnh thoảng khi chúng ta cần. Uncle Bob đã làm khá nhiều thứ trong sự nghiệp của mình. Ông ấy đã giúp viết và ký vào Tuyên ngôn Agile, điều này thực sự đã giúp ngành công nghiệp suy nghĩ lại cách nhìn nhận việc làm việc với các doanh nghiệp và cách lập kế hoạch phần mềm. Chúng ta có thể biết bao nhiêu về một phần mềm trước khi xây dựng nó, và việc đặt cược nhiều vào điều đó khôn ngoan đến mức nào.

Ông ấy đã giúp chúng ta nhìn nhận cách tổ chức code của mình, từ cách đặt tên cho biến và phương thức đó, cho đến cách tổ chức các phần của ứng dụng và làm cho chúng giao tiếp với nhau. Chúng ta sẽ nói chuyện tối nay, hoặc ông ấy sẽ nói chuyện tối nay, về một chỗ ngay giữa về cách bạn tổ chức các đối tượng của mình và làm cho chúng liên quan đến nhau.

Ông ấy đã giúp chúng ta hiểu các ngôn ngữ lập trình mới và tại sao chúng quan trọng và tại sao chúng giúp ích. Ông ấy đã viết sách về tất cả những điều này dọc đường và nói chuyện tại các sự kiện như thế này khắp nơi. Chúng ta nợ ông ấy rất nhiều. Ông ấy đã nâng cao trình độ chung của lĩnh vực chúng ta khá nhiều. Ông ấy đã ghi chép lại tất cả dọc đường. Vì vậy, hy vọng tôi không đã tô vẽ ông ấy quá nhiều, nhưng với điều đó, Bob...

Được rồi. Để xem. Tôi có cần bật công tắc không? Hay là người âm thanh đang làm điều này? Bật công tắc? Vâng. Được rồi. Ồ, tôi nói nhỏ đấy. Thế này được không? Ồ, tốt. Okay.

## Nước: Một Phép Ẩn Dụ để Hiểu về Sự Phụ Thuộc

Có bao nhiêu người trong các bạn là lập trình viên? Nhìn kìa. Vâng, lập trình viên. Có ai không phải không? Bạn không phải? Bạn là gì? Sản phẩm trực tuyến? Đó không phải là lập trình viên. Không, không phải. Okay. Được rồi. Đa số các bạn là.

Nước là gì? Một chất tự nhiên. Tốt. Định nghĩa rất chung. Mềm hay cứng? Bản thân nước thì mềm. Chính những thứ trong đó làm cho nó cứng. Công thức hóa học là gì? H₂O. Nghĩa là hai hydro, một oxy. Phân tử trông như thế nào? Chuột Mickey. Rất tốt.

Vậy một phân tử nước có một nguyên tử oxy lớn và hai nguyên tử hydro, giống như Chuột Mickey. Góc ở đây, nếu tôi nhớ không nhầm, là 103 độ vì những lý do cơ học lượng tử liên quan đến trời biết là gì. Tại sao ba nguyên tử này gắn với nhau?

Bây giờ, trước khi bạn đưa cho tôi câu trả lời về liên kết cộng hóa trị, hãy nghĩ về những nguyên tử đó là gì. Đó là một proton cho hydro, với điện tích dương, và một proton khác cho hydro kia với điện tích dương. Có bao nhiêu proton trong nguyên tử oxy? Tám. Tám proton ở trung tâm hạt nhân oxy. Sau đó xung quanh những proton đó là đám mây electron này: một electron cho hydro, một electron cho hydro kia, và tám cho oxy.

Đám mây electron đó có điện tích âm. Vậy ba nguyên tử này lẽ ra phải đẩy nhau vì chúng được bao quanh bởi điện tích âm. Tại sao chúng gắn với nhau? Cái gì giữ chúng? Chúng chia sẻ. Đó là kinh điển. Thực sự chắc chắn. Chúng chia sẻ cái gì?

Hãy nhìn kỹ hơn bây giờ. Trước hết, nếu nhiệt độ đủ thấp, nếu những nguyên tử này không di chuyển đủ nhanh, chúng sẽ đẩy nhau. Chúng sẽ không đến gần nhau. Bạn phải làm cho chúng di chuyển nhanh. Bạn phải làm cho chúng va vào nhau. Nếu bạn làm cho chúng va vào nhau, một điều thực sự thú vị xảy ra, đó là:

Trước hết, bạn có điện tích dương ở đây, điện tích dương ở đây, và điện tích dương ở đây. Các electron muốn đi đâu? Chúng muốn ở càng gần các điện tích dương càng tốt, điều đó xảy ra ngay tại đây. Vì vậy bạn có một ưu thế của điện tích âm giữa các proton, và điện tích âm giữ các proton lại với nhau. Bạn chỉ cần đưa chúng đủ gần, và sau đó chúng kiểu như trượt, và chúng khớp lại với nhau.

Bây giờ, bạn đã đề cập rằng chúng chia sẻ. Vâng, chúng có chia sẻ. Tại sao? Bởi vì có một số hiệu ứng cơ học lượng tử kỳ lạ liên quan đến Nguyên lý Loại trừ Pauli, nói rằng lớp vỏ ngoài của nguyên tử oxy có thể chứa tới tám electron trong đó. Nó chỉ có sáu, và hai electron từ các hydro sẽ chỉ khớp vào chỗ đó. Đó không phải là lý do chúng gắn với nhau. Chúng gắn với nhau vì điện tích này. Chúng sẽ khớp vì hiệu ứng cơ học lượng tử. Đó là một liên kết cộng hóa trị.

Một liên kết cộng hóa trị xảy ra khi bạn xoay sở để có phần lớn điện tích âm giữ các điện tích dương lại với nhau. Tất nhiên các electron bay lượn ở đây, và đó là một thứ xác suất lớn. Chúng thực sự ở khắp nơi, nhưng chúng sẽ dành nhiều thời gian hơn ở đó, và chúng giữ toàn bộ thứ lại với nhau.

Bây giờ nhìn vào đây. Tôi chia đôi phân tử, cắt đầu của Chuột Mickey làm đôi. Đa số điện tích âm ở đâu? Nó ở trên đường. Phân tử không có điện tích phân bố đều. Nó âm hơn ở trên đường. Nó dương hơn ở dưới đường. Phân tử có một lưỡng cực, một điện tích xuyên qua nó.

Vợ tôi tức giận với tôi khi tôi thêm 'T' vào cuối "across." Điều đó có nghĩa là phân tử muốn xoay và gắn vào bất cứ thứ gì có điện tích, bất kể điện tích đó là gì.

Tại sao nước dính vào tay bạn? Tại sao nước làm ướt? Bởi vì tất cả các phân tử xoay và tìm một số điện tích nhỏ trên tay bạn, và chúng dính vào nó một cách tĩnh điện.

Tại sao nước hòa tan được mọi thứ? Bởi vì bất cứ thứ gì có một chút điện tích, phân tử nước sẽ dính vào.

Tại sao nước tốt cho việc rửa? Bởi vì nó có thể dính vào bụi bẩn.

Ai đã làm trò nhỏ dễ thương này: bạn lấy một vòi nước và bạn tạo một dòng nước thực sự mỏng? Bạn phải làm cho nó mỏng nhất có thể, nhưng không có giọt. Nó phải là một dòng nước mỏng đẹp. Sau đó bạn lấy một quả bóng bay và bạn chà quả bóng bay vào đầu của bạn. Bạn lấy quả bóng bay và bạn đặt nó ở đó, và nước chỉ "hui!" Hút. Quả bóng bay hút rất nhiều. Ai đã làm cái đó? Làm cho con bạn ngạc nhiên. Các con, xem này!

Tất nhiên, đó không phải là những gì chúng ta nên nói chuyện, phải không? Vậy chúng ta sẽ phải từ bỏ chủ đề thú vị này và nói về một cái gì đó tầm thường hơn: các nguyên tắc SOLID của thiết kế agile và hướng đối tượng. Tôi có thể ném một đống tính từ khác vào đó nếu bạn muốn.

## Vấn Đề: Sự Tăng Trưởng Theo Cấp Số Nhân của Lập Trình Viên

Điều gì sai với phần mềm? Có bao nhiêu người trong các bạn đã là lập trình viên trong hơn một năm? Năm năm? Chú ý cách đó là một nửa. Mười năm? Chú ý cách chúng ta đã cắt nó làm đôi một lần nữa. Mười lăm năm? Một nửa lần nữa. Tại sao? Điều đó thú vị.

Dân số lập trình viên là bao nhiêu? Có bao nhiêu lập trình viên trên thế giới? Rất nhiều. Nó phụ thuộc vào việc bạn có đếm các lập trình viên VBA hay không, nhưng rất nhiều. Nó có lẽ gần 100 triệu, tùy thuộc vào cách bạn đếm.

Tôi bắt đầu lập trình cách đây hơn 40 năm. Có bao nhiêu lập trình viên vào năm 1970? Ồ, không quá 50.000. Có lẽ là khoảng 50.000, nhưng không phải 100 triệu. Nếu bạn quay lại 10 năm trước đó, năm 1960, có bao nhiêu lập trình viên vào năm 1960? Vài trăm người trên thế giới. Vài trăm. Hầu như không có. Họ thậm chí không phải là lập trình viên. Họ là các nhà phát triển phần cứng bị lôi kéo vào việc viết code vì không ai biết cái quái gì thứ này là gì.

Bạn nghĩ về sự tiến triển theo thời gian này. Chúng ta bắt đầu với một vài lập trình viên vào những năm 50, xoay sở có được vài trăm vào những năm 60, vài nghìn vào những năm 70. Bây giờ chúng ta gần 100 triệu. Đó là một sự tăng trưởng hình học, theo cấp số nhân—bạn nói cho tôi từ đó là gì—. Nó có một tốc độ tăng gấp đôi. Tốc độ tăng gấp đôi phải là năm năm. Cứ mỗi năm năm, dân số lập trình viên tăng gấp đôi.

Điều đó có một hàm ý thú vị: một nửa số lập trình viên có ít hơn năm năm kinh nghiệm. Điều này luôn đúng. Những người già như tôi—người ta tự hỏi, "Sao không có nhiều lập trình viên lớn tuổi hơn? Chắc tất cả đều nghỉ việc. Chắc tất cả đều đi nuôi gà hay gì đó." Chúng tôi không nghỉ việc. Chỉ là không có nhiều chúng tôi thôi. Chúng tôi vẫn ở đây hết. Chúng tôi vẫn đang viết code. Chỉ là có rất ít chúng tôi ngay từ đầu.

Làm thế nào chúng ta đối phó với thực tế trong ngành của chúng ta rằng chúng ta đang bị mắc kẹt trong một đường cong theo cấp số nhân đảm bảo sự thiếu kinh nghiệm vĩnh viễn? Một nửa số lập trình viên sẽ có ít hơn năm năm kinh nghiệm, và điều đó sẽ vẫn đúng cho đến khi đường cong theo cấp số nhân đó giảm dần, có lẽ sẽ xảy ra trong vài năm tới, có thể vài thập kỷ nữa. Nhưng sớm thôi. Làm thế nào chúng ta đối phó với điều đó? Vấn đề với điều đó là gì?

Vấn đề với điều đó? Ồ, để tôi hỏi câu hỏi này: có bao nhiêu người trong các bạn, những người già, đã bị làm chậm lại bởi code thực sự tệ? Những người trẻ không giơ tay lên. Có bao nhiêu người trong các bạn, những người trẻ, đã bị làm chậm lại bởi code thực sự tệ mà bạn đã viết ngày hôm qua? Vâng.

Chúng ta biết code tệ làm chậm chúng ta. Tại sao chúng ta viết nó? Tại sao chúng ta viết code làm chậm chúng ta? Nó làm chậm chúng ta bao nhiêu? Rất nhiều. Mỗi lần chúng ta chạm vào các module, chúng ta lại bị làm chậm. Mỗi lần chúng ta chạm vào code tệ, nó làm chậm chúng ta.

Tại sao chúng ta viết những thứ làm chậm chúng ta này? Câu trả lời cho điều đó là, "Ồ, chúng ta phải đi nhanh." Tôi sẽ để bạn giải quyết sự mâu thuẫn logic ở đó.

## Nguyên Tắc Chính: Đi Chậm để Đi Nhanh

Thông điệp của tôi hôm nay với các bạn là: bạn không đi nhanh bằng việc viết rác. Bạn không đi nhanh bằng việc vội vàng. Bạn không đi nhanh bằng việc xé toạc code và chỉ làm cho nó hoạt động và phát hành nó nhanh nhất có thể. Bạn muốn đi nhanh? Bạn làm một công việc tốt. Bà của bạn đã nói với bạn điều này từ lâu rồi. Bạn muốn đi nhanh? Bạn làm tốt. Bạn dành thời gian. Bạn nghiên cứu vấn đề. Bạn di chuyển cẩn thận thay vì nhanh chóng.

Một lập trình viên đi nhanh, một lập trình viên đang vội vàng như điên, được đảm bảo sẽ đi chậm. Một lập trình viên ngồi cẩn thận và suy nghĩ về nó và gõ một vài ký tự và nhìn vào code và làm sạch nó một chút và làm cái này và cái kia—anh ta sẽ đi nhanh. Không quan trọng là cô ấy hay anh ấy.

Hãy tưởng tượng điều này: bạn đang có trải nghiệm ngoài cơ thể, lơ lửng phía trên một bàn mổ nơi một bác sĩ phẫu thuật đang thực hiện phẫu thuật tim hở trên bạn. Bạn đang nhìn xuống bác sĩ phẫu thuật đó khi anh ta đang mở ngực bạn và cố gắng sửa chữa tổn thương cho tim bạn. Bạn muốn bác sĩ phẫu thuật này cư xử như thế nào? Cẩn thận. Thận trọng.

Bây giờ, anh ta có một hạn chọt—theo nghĩa đen. Nhưng bạn hy vọng, ngay cả vì hạn chót này, ngay cả khi đối mặt với hạn chót này, bạn hy vọng bác sĩ phẫu thuật này đang cư xử cẩn thận, bình tĩnh, thận trọng, tuân theo các kỷ luật của mình, đi gần theo sách—không chính xác theo sách, nhưng gần theo sách—bình tĩnh yêu cầu một dụng cụ, bình tĩnh làm một việc, hành động như bác sĩ phẫu thuật biết họ đang làm gì.

Bạn không muốn bác sĩ phẫu thuật hành động như một lập trình viên phần mềm điển hình. Hãy nghĩ về điều đó một lúc.

## Các Triệu Chứng của Phần Mềm Tệ

Điều gì sai với phần mềm? Ồ, đó là những gì sai với nó. Nhưng chúng ta có thể cụ thể hơn. Các triệu chứng của phần mềm tệ là gì? Làm thế nào bạn biết bạn có code tệ? Nó làm gì với bạn, ngoài điều hiển nhiên là làm chậm bạn? Nó làm chậm bạn như thế nào?

Nó khó hiểu. Okay, được rồi. Bạn nhìn vào nó và bạn nói, "Hả." Bây giờ, đó là code tệ. Bạn nhìn vào code và bạn nói, "Đó là code tệ," bởi vì code tốt nên giải thích những gì nó đang làm. Với code tốt, bạn nên nhìn vào nó và bạn nên nói, "Ồ vâng. Ồ vâng." Nó nên nhàm chán. Bạn nên đọc nó và nói, "Hoàn toàn rõ ràng." Đó là code tốt.

Code tệ? Trời. Nó đang làm gì khác? Điều gì xảy ra khi bạn sửa đổi code tệ? Bạn phá vỡ cái gì đó. Đó là cách bạn biết đó là code tệ. Nếu bạn sửa đổi cái gì đó và điều đó phá vỡ cái gì đó khác, code là tệ.

### Tính Cứng Nhắc (Rigidity)

Một trong những triệu chứng được gọi là **tính cứng nhắc**. Tính cứng nhắc là khi bạn chạm vào code và bây giờ bạn phải sửa đổi một lượng lớn code khác để quay lại sự nhất quán với sửa đổi này mà bạn đã thực hiện.

Sếp của bạn đến với bạn một ngày và nói, "Bạn có thể sửa lỗi này không?" Bạn nhìn vào báo cáo lỗi và bạn nói, "Hừm, tôi biết chính xác lỗi này ở đâu." Bạn không nói với sếp của bạn điều đó. Bạn chỉ nghĩ thầm, "Tôi biết chính xác lỗi này ở đâu. Tôi thực sự đã đưa nó vào code ngày hôm qua." Vì vậy bạn nói với sếp của bạn, "Ba tuần," bởi vì đó là ước tính tối thiểu bạn không được phép đưa ra. Một ước tính ít hơn ba tuần? Tất cả các lập trình viên khác đến cubicle của bạn với gậy.

Sếp đi mất vui vẻ. Anh ta đã nhận được ước tính tối thiểu. Anh ta sẽ quay lại trong ba tuần. Bây giờ bạn đến module vì bạn biết lỗi này ở đâu. Bạn đến module. Nó hiện trên màn hình của bạn. Bạn nhìn vào nó một phút và nói, "Vâng, đó là lỗi. Tôi biết cách sửa nó. Để tôi làm—" Bạn biết đấy. "Khoan đã. Nếu tôi làm điều đó, có module khác ở đây gọi hàm đó. Tốt hơn tôi nên kiểm tra nó."

Bạn mở module đó lên. "Ồ trời, tôi sẽ phải làm việc với cái đó một chút. Nó thay đổi một chút. Ồ khoan, nếu tôi thực hiện thay đổi đó, có những module ở đây..." Và bây giờ bạn bắt đầu đuổi theo đuôi qua code. Các tuần trôi qua: hai tuần, ba tuần, bốn hoặc năm. Sếp của bạn quay lại và nói, "Tôi nghĩ bạn sẽ hoàn thành cái này trong ba tuần."

"Tôi sẽ hoàn thành vào ngày mai, tôi thề!"

Đến khi bạn hoàn thành, bạn đã chạm vào mọi module trong hệ thống. Sếp của bạn nói, "Cái quái gì mất nhiều thời gian như vậy?"

Và bạn phát ngôn những từ bất tử của mọi lập trình viên phần mềm: "Ồ, nó phức tạp hơn tôi nghĩ nhiều."

Đó là code cứng nhắc. Code có các phụ thuộc quanh co theo nhiều hướng đến mức bạn không thể thực hiện một thay đổi cô lập mà không thay đổi mọi thứ khác xung quanh nó. Các phụ thuộc tệ. Các hệ thống được ghép nối chặt chẽ.

### Tính Dễ Vỡ (Fragility)

Một triệu chứng khác của code tệ, tương tự như cái đó, được gọi là **tính dễ vỡ**. Tính dễ vỡ là xu hướng code bị vỡ ở nhiều nơi, ngay cả khi bạn chỉ thay đổi nó ở một nơi. Bạn thực hiện một thay đổi rất đơn giản, và một đống thứ khác bị vỡ. Nhưng chúng vỡ ở các phần của code không có mối quan hệ gì với những gì bạn đã thay đổi.

Bạn thực hiện một thay đổi nhỏ ở đây, và cái gì đó ở kia không hoạt động nữa. Bạn thực hiện một thay đổi đối với cách tính lương cho nhân viên theo giờ—chỉ một thay đổi nhỏ—và bây giờ nó sẽ không in báo cáo cho sếp công đoàn. Nó bị crash khi in báo cáo đó. Tại sao điều đó xảy ra?

Đây là code dễ vỡ. Code dễ vỡ vỡ theo những cách kỳ lạ và kỳ quặc, những cách mà bạn không thể dự đoán. Bạn thực hiện một thay đổi ở đây, cái gì đó ở kia vỡ. Bạn không dự đoán nó. Bạn không biết tại sao nó xảy ra. Bạn phải đuổi bắt lỗi xung quanh. Cuối cùng, bạn nhận ra, "Ồ vâng, hàm ở kia đã đặt một cờ, và người kia ở kia đã sử dụng cờ đó."

Xe của bạn có một cửa sổ điện không hoạt động. Bạn đưa nó đến một thợ máy. Thợ máy nhìn vào nó một lúc, nói, "Vâng, tôi có thể sửa cái đó." Bạn quay lại vào ngày hôm sau. Thợ máy tự hào cho bạn thấy cửa sổ điện hoạt động. Bạn cảm ơn anh ta. Bạn vào xe. Bạn bật nó lên. Xe không khởi động.

Bạn sẽ không quay lại với thợ máy đó. Thợ máy đó là một thằng ngốc. Và đó là những gì xảy ra khi các quản lý và khách hàng thấy các lập trình viên phần mềm thay đổi một thứ ở đây và cái gì đó ở kia vỡ. Không có gì tạo ra nỗi sợ hãi nhiều hơn vào trái tim của khách hàng hay quản lý so với cái đó, bởi vì đột nhiên, các quản lý và khách hàng, những người không có ý tưởng gì về công nghệ, họ tin rằng những người này đã mất kiểm soát.

Họ bắt đầu nghĩ rằng nếu họ nhìn dưới nắp capo, nó sẽ là rác. Họ đúng. Họ bắt đầu nghi ngờ rằng có lẽ chất lượng của sản phẩm này không tốt chút nào. Họ trở nên sợ hãi khi để bạn thực hiện thay đổi.

Có ai làm việc tại một công ty nơi quản lý cuối cùng đã nói, "Được rồi, không ai thay đổi module đó"? Module đó quá dễ vỡ. Mỗi lần bạn chạm vào nó, cái gì đó khác bị sai. Không ai chạm vào module đó. Đó là thất bại cuối cùng của một lập trình viên phần mềm, khi doanh nghiệp, những người không có năng lực kỹ thuật, nói với bạn bạn không được chạm vào module đó.

### Tính Bất Động (Immobility)

Triệu chứng thứ ba của code tệ: sếp của bạn đến với bạn và nói, "Bạn biết module bạn phải viết đó không? Bạn biết đấy, Joe, ở nhóm khác kia, anh ta đã viết một cái giống như thế năm ngoái. Bạn nên đến nói chuyện với Joe và xem bạn có thể sử dụng một số code của anh ta không."

Ồ, bạn biết Joe, và bạn biết loại code mà Joe viết, và bạn không muốn đến đó và nói chuyện với Joe. Nhưng sếp của bạn đã bảo bạn làm điều đó. Vì vậy bạn đến đó và bạn nhìn. Chắc chắn rồi, module của Joe làm chính xác những gì bạn cần nó làm. Nhưng nó làm nhiều hơn thế. Nó ghép nối với một số framework kỳ lạ. Nó sử dụng một số cơ sở dữ liệu kỳ quặc. Bạn càng nhìn vào nó, bạn càng nhận ra rằng vâng, bạn có thể sử dụng code của Joe, nhưng các vấn đề mà bạn sẽ mang vào nghiêm trọng đến mức cuối cùng bạn nói, "Sẽ dễ dàng hơn cho tôi nếu tự viết nó."

Các phần mong muốn của code được ghép nối kinh khủng với các phần không mong muốn của code đến mức bạn không thể sử dụng các phần mong muốn của code ở nơi khác. Code của Joe là rác.

## Nguyên Nhân Gốc Rễ: Sự Ghép Nối (Coupling)

Làm thế nào chúng ta đối phó với điều này? Điểm chung của tất cả những khiếm khuyết đó là gì?

Ồ, được rồi, spaghetti là từ mà chúng ta sử dụng để nói, "Ồ, code này rối tung lên hết." Nhưng có một thuật ngữ kỹ thuật hơn: **sự ghép nối** (coupling).

Lý do code cứng nhắc là vì các module phụ thuộc vào nhau theo những cách không mong muốn. Lý do code dễ vỡ là vì code phụ thuộc vào các cấu trúc dữ liệu theo những cách không mong muốn. Lý do tôi không thể lấy các phần mong muốn trong code của Joe và sử dụng chúng trong hệ thống của tôi là vì code của Joe phụ thuộc vào code khác theo những cách không mong muốn.

Điểm chung ở đây là sự ghép nối, phụ thuộc. Có thể nói, và tôi nghĩ là chính xác, rằng **phần lớn thiết kế phần mềm là quản lý các phụ thuộc**—tìm ra nơi đặt code và cắt các phụ thuộc sao cho các phụ thuộc không chạy theo những hướng kỳ lạ và kỳ quặc.

Làm thế nào bạn làm điều đó?

Làm thế nào bạn làm điều đó?

## Hiểu về Các Phụ Thuộc Mã Nguồn (Source Code Dependencies)

Có hai người trên thế giới mà tôi muốn nói chuyện: những người thiết kế bảng trắng. Bạn biết đấy, phấn không làm cái này. Chúng ta có chỗ ở đây. Phấn cũng không làm cái đó.

Hãy nói rằng tôi có một chương trình chính. Hãy nói rằng chúng ta đang viết code bằng C, khoảng năm 1971. Tôi có một chương trình chính. Chương trình chính của tôi gọi một số hàm cấp cao, các module cấp cao. Các module cấp cao đó gọi các module cấp trung. Nhiều hơn một. Có một loạt các module cấp trung này. Các module cấp cao gọi các module cấp trung, và các module cấp trung gọi các module cấp thấp hơn.

Bạn có thể thấy rằng có một cây gọi hàm đẹp đẽ này. Đây là cây gọi hàm thủ tục mà mọi ứng dụng đều có, bất kể ứng dụng là gì. Mọi ứng dụng đều có một trong số này. Chúng ta bắt đầu từ main, và chúng ta bùng nổ xuống dưới hướng tới các hàm cấp thấp nhất trong hệ thống.

Flow của điều khiển đi theo hướng đó. Main gọi cấp cao. Cấp cao gọi cấp trung. Cấp trung gọi cấp thấp. Flow của điều khiển đi xuống dưới.

Module nào trong số này biết về module kia? Thực ra, hãy để tôi làm cho nó đơn giản hơn. Tôi có một module M. Tôi có một module N khác. Module M có một hàm tên là F. M gọi F trong N. Module nào trong số này biết về module kia?

Flow của điều khiển đi theo hướng đó. M biết về N. Làm thế nào chúng ta biết rằng M biết về N? Nếu N thay đổi, M phải... Điều đó có nghĩa là trình biên dịch biết. Làm thế nào trình biên dịch biết rằng M phụ thuộc vào N?

Có một câu lệnh trong mã nguồn của M nói rằng—chúng ta đang làm C bây giờ—`#include n.h`. Nếu là Java, nó sẽ là `import n`. Nếu là .NET, nó sẽ là `using n`. Nhưng dù là ngôn ngữ nào, tên N xuất hiện trong M. Có một phụ thuộc mã nguồn từ M đến N, mà tôi sẽ vẽ bằng một đường màu đỏ.

Chú ý rằng phụ thuộc mã nguồn (đường màu đỏ) và flow của điều khiển (đường màu xanh lá cây) chỉ cùng một hướng. Điều đó là phổ quát. Để một module gọi module khác, bạn phải có một module biết về module kia.

Vậy các phụ thuộc mã nguồn chạy như thế nào ở đây trong cây gọi thủ tục này? Ồ, chúng phải đi như thế này. Các module cấp cao biết về các module cấp thấp.

Hãy nghĩ về điều đó một phút. Khoan đã. Các module cấp cao biết về các module cấp thấp. Quy tắc đó vi phạm cái gì? Sự đảo ngược của... cái gì đó?

Bạn có muốn chính sách cấp cao của mình bị ô nhiễm bởi chi tiết cấp thấp không? Đây là điều làm cho code khó đọc. Bạn đang đọc code, và bạn đang cố gắng hiểu ý tưởng về những gì đang xảy ra ở cấp cao, và đột nhiên bạn đang xử lý các bộ đệm chuỗi, và bạn đang ở một khái niệm cấp thấp thực sự. Chúng ta đang cố gắng ghép lại những gì thực sự đang diễn ra ở cấp cao, nhưng chúng ta không thể, bởi vì các module cấp cao của chúng ta phụ thuộc vào các module cấp thấp, mà lại phụ thuộc vào các module cấp thấp hơn nữa.

Hãy tưởng tượng một số module ở đây có một thứ cực kỳ cấp thấp trong đó—Chúa biết là gì—một số biến cấp thấp mà chúng ta cần thực hiện thay đổi. Tôi thực hiện thay đổi đối với module đó. Ai phải biên dịch lại? Bạn phải theo các mũi tên ngược lại. Mọi module phụ thuộc vào module cấp thấp này sẽ phải biên dịch lại và được triển khai lại. Một thay đổi đối với một chi tiết ảnh hưởng đến chính sách cấp cao, và điều đó là sai.

Nếu bạn nghĩ về điều đó trong một khoảng thời gian nào đó, bạn nghĩ rằng điều đó thật điên rồ. Tôi không muốn các chi tiết thay đổi chính sách cấp cao. Tôi muốn chính sách cấp cao của mình được miễn nhiễm khỏi các chi tiết.

Tại sao code trở nên cứng nhắc? Bởi vì chúng ta có tất cả sự ghép nối này hướng xuống chi tiết.

Tại sao code trở nên dễ vỡ? Bởi vì tôi có thể thực hiện một thay đổi nhỏ ở đây và làm vỡ một loạt thứ ở kia.

Tại sao tôi không thể tái sử dụng một chút chính sách cấp cao? Bởi vì nó được ghép nối chặt chẽ với tất cả những thứ cấp thấp tệ hại này.

## Thiết Kế Hướng Đối Tượng: Giải Pháp

OO là gì? Đối tượng là gì? Thiết kế hướng đối tượng là gì? OO là gì? Tại sao OO là một phần của mọi ngôn ngữ mà bạn làm việc ngày nay?

Ai đang làm việc với Java? Một số người. Tốt. Còn C# thì sao? Thêm một số người nữa. Ruby? Rails? Vâng. Chúng ta sẽ nói về Rails. Còn gì nữa? Chúng ta có những ngôn ngữ nào khác? Python? Vâng. Ruby? Objective-C? Ai đang làm Objective-C, chinh phục thế giới với các ứng dụng iPhone?

Objective-C bao nhiêu tuổi? Nó được phát minh khi nào? 1980.

Ai phát minh ra nó? Brad Cox.

Tại sao anh ta phát minh ra nó? Anh ta là một lập trình viên Smalltalk. Ai đó bắt anh ta lập trình bằng C. Anh ta ghét nó. Anh ta viết một bộ tiền xử lý nhỏ trước C, cho nó một số thuộc tính Smalltalk, gọi nó là Objective-C. Anh ta thành lập công ty riêng của mình, gọi nó là StepStone. Anh ta nghĩ rằng mình sẽ chiếm lĩnh thế giới.

Anh ta đã làm được, trong vài năm. Đây là lựa chọn duy nhất mà các lập trình viên C có để thực hiện bất kỳ loại công việc OO nào. Tất cả chúng ta đều làm việc điên cuồng với Objective-C từ năm 1980 đến 1985. Sau đó Bjarne Stroustrup xuất hiện và xuất bản cuốn sách tuyệt vời này có tên _The C++ Programming Language_. Nó trông giống Kernighan và Ritchie đến mức tất cả các lập trình viên C đã bỏ bất cứ thứ gì họ đang làm và ngay lập tức chấp nhận C++. StepStone tội nghiệp đã phá sản, và ngôn ngữ đó sẽ chết ngày hôm nay nếu không có một tai nạn lịch sử.

Tai nạn lịch sử đó là Steve Jobs, người đã quyết định thuê một doanh nhân thực sự cho Apple. Anh ta đã đi tìm ai? Anh ta đã lấy CEO của Pepsi-Cola, John Sculley—như thể CEO của Pepsi-Cola biết bất cứ điều gì về một công ty máy tính công nghệ cao. Điều mà CEO của Pepsi-Cola biết là đủ chính trị để sa thải Steve Jobs.

Steve Jobs rời đi với rất nhiều tiền. Anh ta có hàng đống tiền, vì vậy anh ta không thực sự quan tâm. Anh ta bắt đầu một công ty mới. Công ty mới được gọi là NeXT. Công ty này bán phần cứng—máy tính màu đen, rất đẹp. Tôi có một cái trong tầng hầm của mình.

Anh ta thuê một loạt lập trình viên đang tìm việc vì ngôn ngữ của họ đã biến mất khỏi họ. Họ tình cờ là các lập trình viên Objective-C. Họ viết hệ điều hành cho máy NeXT. Hệ điều hành được gọi là NeXTSTEP.

Công ty là một thất bại khủng khiếp. Máy không bao giờ làm được gì. Phần mềm không bao giờ đi đến đâu. Điều đó không quan trọng, bởi vì Apple quay lại với Steve bằng tay và đầu gối của họ: "Xin hãy quay lại, Steve. Gã này đang giết chúng tôi."

Steve nói, "Được thôi, nhưng tôi sẽ mang đội của tôi theo." Steve quay lại Apple, mang theo tất cả các lập trình viên Objective-C này với họ, và họ bắt đầu làm việc trên iPod. Đó là lý do tại sao bạn đang lập trình bằng Objective-C—ngôn ngữ tệ nhất có thể cho iPhone, nhân tiện. Bạn không nên làm việc với ngôn ngữ khủng khiếp đó, nhưng okay, đó là tai nạn lịch sử.

Tại sao tất cả những ngôn ngữ này là ngôn ngữ OO? Nó không từng như vậy. Chúng ta từng làm việc với Fortran hoặc PL/1 hoặc COBOL hoặc C. Không ai biết về OO. Tại sao tất cả các ngôn ngữ của chúng ta bây giờ là ngôn ngữ OO? Điều gì hay về OO mà tất cả các ngôn ngữ của chúng ta bây giờ là ngôn ngữ OO?

"Giúp chúng ta tổ chức code." "Như thế nào?" "Đóng gói."

### Ba Trụ Cột: Đóng Gói, Kế Thừa, Đa Hình

Bum-bum-bum. Đóng gói. Vậy, ba từ ma thuật: **đóng gói** (encapsulation), **kế thừa** (inheritance), **đa hình** (polymorphism). Đó là ba từ ma thuật của OO. Mọi ngôn ngữ OO phải được đóng gói, kế thừa và đa hình.

Chúng ta có đóng gói khi chúng ta lập trình bằng C không? Có ai là lập trình viên C cũ ở đây không? Okay, tốt, thưa ngài. Chúng ta có đóng gói không? Chúng ta có đóng gói **hoàn hảo** trong C.

Tất cả những gì bạn phải làm là khai báo trước các hàm và cấu trúc dữ liệu của bạn để bạn không phải implement chúng. Bạn sẽ khai báo trước chúng trong một file header, và sau đó bạn sẽ implement chúng trong một file C. Người dùng của bạn sẽ `#include` file header của bạn. Họ không thể thấy gì về implementation của bạn. Đóng gói hoàn hảo. Không có cách nào người dùng của bạn có thể thấy bất kỳ giá trị dữ liệu nào của bạn. Tất cả những gì họ có thể thấy là các chữ ký hàm của bạn. Họ có thể thấy tên của các cấu trúc dữ liệu của bạn nhưng không có thành viên nào bên trong các cấu trúc dữ liệu của bạn. Đóng gói hoàn toàn hoàn hảo.

Các đối tượng đã làm hỏng hoàn toàn điều đó. C++ xuất hiện và đặt tất cả các biến vào file header. Điều đó đã làm gì? Ồ! Tất cả các biến đột nhiên hiển thị với mọi người. Để kiểm soát điều đó, chúng ta phải phát minh ra những từ khóa khủng khiếp, hacky này: `public`, `private`, `protected`. Những hack khủng khiếp. Nực cười. Một loại băng dán nào đó trên thực tế là chúng ta từng có đóng gói hoàn hảo, nhưng bây giờ chúng ta có cách tiếp cận lộn xộn này là cố gắng chỉ vào các biến nhất định và nói, "Ồ, cái đó là public, nhưng cái đó là private, nhưng cái đó là protected." Ồ, và chúng ta cũng sẽ phát minh ra những cái khác không có tên.

OO đã không cho chúng ta đóng gói. OO đã làm yếu đi đóng gói. Mọi ngôn ngữ đối tượng ngoài kia làm yếu đi sự đóng gói mà chúng ta từng có.

Vì vậy, khi mọi người nói với bạn rằng OO là một ngôn ngữ được đóng gói, họ đã sai. Nó đã phá hủy sự đóng gói tốt và thay thế nó bằng thứ khủng khiếp, hacky này là làm cho trình biên dịch kiểm tra, "Ồ, bạn không được phép chạm vào biến đó, vì vậy tôi thậm chí sẽ không phản hồi."

Câu trả lời về đóng gói chỉ đơn giản là không có ở đó. Chúng ta đã có đóng gói. Bây giờ chúng ta không có.

### Kế Thừa (Inheritance)

Bạn có thể làm kế thừa trong C không? Vâng, nhưng nó khó. Ồ, chúng ta có các union, và không quá khó để lấy hai cấu trúc dữ liệu và cho chúng các phần tử chung và chỉ thay đổi chúng ở cuối, vì vậy một vài phần tử cuối cùng là khác nhau. Sau đó bạn có thể ép kiểu con trỏ từ cái này sang cái kia và truyền chúng xung quanh bên trong các hàm, giống như các đối tượng đa hình. Chúng ta từng làm điều này mọi lúc. Đó là một cách tiếp cận rất phổ biến trong C.

C++ làm cho nó tiện lợi hơn một chút, nhưng không nhiều. Kế thừa bội rất tiện lợi hơn trong C++. Đó là lý do tại sao họ lấy nó đi trong Java.

Tại sao Java không có kế thừa bội? Khoan, tại sao C# không có kế thừa bội? Bởi vì Java không có nó.

Tại sao Java không có kế thừa bội? "Vấn đề kim cương."

Vấn đề kim cương đã được giải quyết. Vấn đề kim cương rất đơn giản. Về cơ bản bạn có được một dẫn xuất của hai lớp cơ sở. Cả hai lớp cơ sở đó đều có một lớp cơ sở chung. Đây là Kim Cương Chết Người (Deadly Diamond of Death). Có một sự mơ hồ lớn xung quanh điều đó. Chúng ta sẽ không đi vào chi tiết, nhưng tại sao họ không giải quyết nó? Bởi vì nó đã được giải quyết nhiều lần trước đó bởi nhiều ngôn ngữ khác.

Tại sao Java không giải quyết điều này? Để đơn giản hóa trình biên dịch. Đây là dự đoán tốt nhất của tôi. Họ quá lười để xử lý nó, vì vậy họ phát minh ra một số hack khủng khiếp gọi là `interface` và họ dán nó vào ngôn ngữ thay thế. Sau đó tất cả chúng ta cúi đầu và tôn kính interface. "Interface là tốt. Interface là tuyệt vời."

Một interface chẳng qua là một lớp trừu tượng với một loạt các phương thức trừu tượng trong đó. Nó không phải là một thứ đặc biệt gì. Thực tế, điều duy nhất đặc biệt trong Java về một interface là những gì bạn không thể làm với nó. Bạn không thể kế thừa bội—bạn không thể kế thừa nhiều lớp vào một lớp khác. Tại sao? Bởi vì điều duy nhất bạn được phép kế thừa bội là các interface.

Một hack. Một hack lười biếng. Họ không nên làm điều đó. Họ nên giải quyết vấn đề thay vì chỉ ném nó ra ngoài đó. Sau đó điều gì đã xảy ra? Nó đi vào một loạt ngôn ngữ khác.

Bạn có thể nói tôi đang tận hưởng bản thân mình ở đây, phải không?

Okay. Vì vậy, chúng ta đã có kế thừa ở một dạng nào đó trong C. Chúng ta có thể mô phỏng nó, nhưng nó không thuận tiện lắm. Vì vậy, những gì tôi sẽ làm ở đây là tôi sẽ cho C nửa điểm tín dụng. Tôi sẽ cho OO nửa điểm tín dụng vì đã cho chúng ta một sự kế thừa thuận tiện hơn một chút.

Các lập trình viên Ruby, bạn sử dụng kế thừa bao nhiêu? Vâng. Để làm gì? Có lý. Active Record? Bạn có sử dụng kế thừa cho đa hình không? Bạn có cần sử dụng kế thừa cho các kiểu duck không? Không biết?

Trong Ruby, trong Python, trong các ngôn ngữ được gõ động, bạn có thể có đa hình mà không cần bất kỳ kế thừa nào. Thực tế, tất cả đa hình trong các ngôn ngữ này đến mà không cần kế thừa. Kế thừa không cần thiết. Bạn sử dụng kế thừa để làm gì trong một ngôn ngữ được gõ động? Đó là để kế thừa hành vi và biến, nhưng không phải interface. Bạn nhận được tất cả đa hình bạn muốn miễn phí.

Trong Java và .NET và C++, chúng ta phải có kế thừa để làm đa hình. Một trong những điểm yếu của các ngôn ngữ này là bạn phải sử dụng kế thừa để có được đa hình.

### Đa Hình (Polymorphism)

Điều đó để lại cho chúng ta cái cuối cùng trong ba cái đó: đa hình. Chúng ta có đa hình trong C không? Ồ. Có thể. Có thể.

Trong C, chúng ta có thể làm điều này: một ứng dụng copy trong C.

```c
while ((c = getchar()) != EOF) {
    putchar(c);
}
```

Chương trình copy của UNIX, được đơn giản hóa rất nhiều. Chương trình đó làm gì? Ồ, bạn đã làm một điều tồi tệ. Nó làm, nhưng không chính xác. Nó sao chép từ đầu vào chuẩn sang đầu ra chuẩn. `getchar` đọc một ký tự từ đầu vào chuẩn. `putchar` ghi một ký tự ra đầu ra chuẩn.

Đầu vào chuẩn là gì? Nó mặc định là bàn phím, nhưng nó không nhất thiết phải là bàn phím. Nó có thể là bất cứ thứ gì. `putchar` ghi ra đầu ra chuẩn, thường là terminal—tôi sử dụng từ "terminal" cho thấy tôi già như thế nào—nhưng nó không nhất thiết phải là terminal. Nó có thể là bất cứ thứ gì.

Đó là các lời gọi đa hình. `getchar` và `putchar` là các lời gọi phương thức đa hình trên một lớp có tên là `file`, nếu bạn muốn. Chúng giống như bất kỳ hàm ảo nào khác trong C++, giống như bất kỳ hàm đa hình nào khác trong Java.

Tôi có một hệ thống phân cấp kiểu ở đó. Mỗi driver I/O chỉ là một kiểu khác. Tôi có đóng gói hoàn hảo trong C. Tôi có kế thừa hợp lý trong C. Tôi có thể có được đa hình trong C.

Đa hình đó hoạt động như thế nào? Làm thế nào mà flow của điều khiển rời khỏi `getchar` và đi vào driver bàn phím? Điều đó xảy ra như thế nào?

Không, nó khéo léo hơn nhiều so với thế. Khủng khiếp hơn nhiều. Mỗi driver I/O phải có năm hàm ma thuật. Mỗi driver I/O có năm hàm này. Tất cả chúng đều có chữ ký hoàn toàn giống nhau: `read`, `write`, `open`, `close`, `seek`.

Nếu bạn viết một driver I/O, bạn phải implement năm hàm đó, và năm hàm phải có cùng chữ ký. Bạn có `open`, `close`, `read`, `write`, `seek`.

Hệ điều hành sẽ lấy các con trỏ đến các hàm đó và đặt chúng vào một bảng. Khi bạn gọi `getchar`, nó sẽ đi đến bảng cho đầu vào chuẩn và nói, "Hàm read ở đâu? Ồ, tôi sẽ gọi hàm read đó," không biết hàm read là gì, chỉ biết rằng nó có chữ ký đúng.

Các lập trình viên C++, bảng đó là gì? Đó là **vtable**. Chính xác. Cách C++ implement các hàm ảo.

Các lập trình viên Java, bạn có con trỏ đến các hàm không? Không. Tại sao không? Bởi vì bạn có đa hình. Nếu bạn có đa hình, bạn không cần con trỏ đến các hàm.

Các lập trình viên C#, bạn có con trỏ đến các hàm không? Bạn có những thứ delegate xấu xí đó, nhưng chúng không hoàn toàn là con trỏ đến các hàm. Không, bạn không có con trỏ đến các hàm.

Các lập trình viên C++ có, nhưng họ không sử dụng chúng nếu họ được nhìn thấy.

Các lập trình viên Ruby, bạn có con trỏ đến các hàm không? Không thực sự.

Các ngôn ngữ OO không cần con trỏ đến các hàm vì các ngôn ngữ OO là đa hình. Nếu bạn có phân phối đa hình, bạn không cần con trỏ đến các hàm.

Chúng ta không làm đa hình trong C nhiều lắm. Lý do chúng ta không làm nó trong C nhiều lắm là vì nó nguy hiểm như địa ngục. Nếu bạn tạo một bảng các con trỏ hàm, bạn phải tải bảng đó, và bạn phải đảm bảo rằng mọi người gọi các hàm thông qua bảng đó. Nếu ai đó vi phạm bất kỳ quy tắc nào trong số đó, bạn có một vấn đề khủng khiếp trong tay. Vì vậy, phần lớn thời gian, chúng ta không có đa hình trong C hoặc trong bất kỳ ngôn ngữ nào trước C++.

Khi C++ xuất hiện, đột nhiên chúng ta có được đa hình rẻ, dễ dàng, an toàn.

## Sức Mạnh của Đa Hình: Đảo Ngược Phụ Thuộc

Vậy hướng đối tượng đã làm gì cho chúng ta? Nó không cho chúng ta đóng gói—chúng ta đã có nó. Nó không cho chúng ta kế thừa—chúng ta đã có nó. Nó không cho chúng ta đa hình—chúng ta đã có nó.

Nhưng nó đã cho chúng ta đa hình an toàn và tiện lợi. Và đó là một điều lớn. Đó là một điều rất lớn.

Tại sao? Bởi vì đa hình cho phép bạn đảo ngược các phụ thuộc mã nguồn.

Hãy để tôi chỉ cho bạn điều này. Đây là một chương trình đơn giản:

```
main() → function()
```

Main gọi một function. Có một phụ thuộc mã nguồn từ main đến function. Main biết về function. Function không biết về main.

Flow của điều khiển đi từ main đến function. Phụ thuộc mã nguồn đi từ main đến function. Chúng cùng một hướng.

Bây giờ hãy để tôi chỉ cho bạn điều này:

```
main() → interface ← implementation
```

Main gọi một interface. Interface là một lớp trừu tượng hoặc một giao diện Java hoặc một protocol Objective-C. Implementation implements interface đó.

Flow của điều khiển đi từ main đến implementation. Nhưng phụ thuộc mã nguồn? Main phụ thuộc vào interface. Implementation cũng phụ thuộc vào interface.

Phụ thuộc mã nguồn bị đảo ngược. Flow của điều khiển đi một hướng. Phụ thuộc mã nguồn đi hướng khác.

Đó là sức mạnh của hướng đối tượng. Đó là điều mà hướng đối tượng cho chúng ta. Khả năng đảo ngược các phụ thuộc mã nguồn.

Tại sao điều đó quan trọng? Bởi vì bây giờ tôi có thể phá vỡ các chu kỳ. Bây giờ tôi có thể tạo ra các cấu trúc mà các phụ thuộc mã nguồn không tạo thành các vòng lặp.

Bây giờ tôi có thể tạo ra các kiến trúc mà các module cấp cao không phụ thuộc vào các module cấp thấp. Các module cấp thấp phụ thuộc vào các module cấp cao.

Đó là sức mạnh của hướng đối tượng. Đó là lý do tại sao hướng đối tượng quan trọng. Không phải vì đóng gói. Không phải vì kế thừa. Vì đa hình. Và cụ thể, vì khả năng đảo ngược các phụ thuộc.

## Các Nguyên Tắc SOLID

Bây giờ, làm thế nào chúng ta sử dụng sức mạnh này? Làm thế nào chúng ta sử dụng khả năng đảo ngược các phụ thuộc để tạo ra thiết kế tốt hơn? Đó là những gì các nguyên tắc SOLID nói về.

SOLID là một từ viết tắt. Nó đại diện cho năm nguyên tắc thiết kế hướng đối tượng. Năm nguyên tắc này giúp bạn quản lý các phụ thuộc. Chúng giúp bạn tạo ra các cấu trúc mà các module ít ghép nối hơn, linh hoạt hơn, dễ thay đổi hơn.

Năm nguyên tắc là:

- **S** - Single Responsibility Principle (Nguyên tắc Trách nhiệm Đơn)
- **O** - Open/Closed Principle (Nguyên tắc Mở/Đóng)
- **L** - Liskov Substitution Principle (Nguyên tắc Thay thế Liskov)
- **I** - Interface Segregation Principle (Nguyên tắc Phân tách Giao diện)
- **D** - Dependency Inversion Principle (Nguyên tắc Đảo ngược Phụ thuộc)

Chúng ta sẽ đi qua từng nguyên tắc một.

### S - Nguyên Tắc Trách Nhiệm Đơn (Single Responsibility Principle - SRP)

Nguyên tắc đầu tiên trong số này được gọi là **Nguyên tắc Trách nhiệm Đơn** (Single Responsibility Principle). Nguyên tắc Trách nhiệm Đơn nói rằng: **một lớp nên chỉ có một lý do để thay đổi**. Nó nên có một trách nhiệm duy nhất để thay đổi.

Đừng nhầm lẫn về từ "trách nhiệm" (responsibility). Từ "trách nhiệm" không có nghĩa là "chức năng" (function). Không có nghĩa là "những gì nó làm". Từ "trách nhiệm" có nghĩa là "thay đổi" (change). Nó có một lý do để thay đổi.

Nhìn vào lớp đó ở trên kia, lớp `Employee` đó. Nó có một số phương thức trên đó: `calcPay`, `reportHours`, `writeEmployee`. `calcPay` tính toán lương của một nhân viên. Đó là một ứng dụng bảng lương. `reportHours` tạo ra một số báo cáo khủng khiếp mà các kiểm toán viên sẽ đọc theo thời gian về các nhân viên. `writeEmployee` là một hàm cơ sở dữ liệu lưu nhân viên vào cơ sở dữ liệu.

Lớp này có bao nhiêu lý do để thay đổi? Có bao nhiêu thay đổi có thể được yêu cầu? Có bao nhiêu nguồn thay đổi?

Hãy nghĩ về điều đó. Nếu `calcPay` tốn của công ty một triệu đô la, giám đốc điều hành cấp C nào sẽ phát hiện ra điều đó đầu tiên? Đó sẽ là CFO, CEO, hay CTO?

Đó sẽ là CFO vì anh ta sẽ là người nhìn vào sổ sách, vào các tài khoản. Sau đó anh ta sẽ đến với các lập trình viên và sa thải họ.

Nếu có một lỗi trong hàm `reportHours` khiến công ty mất một triệu đô la, giám đốc điều hành cấp C nào sẽ phát hiện ra điều đó đầu tiên? CFO, CEO, hay CTO?

CEO, bởi vì ông ta kiểm soát các kiểm toán viên kiểm tra các báo cáo về những gì đã được thực hiện và những gì đã được chi tiêu. Ông ta sẽ đến với các developer và sa thải họ.

Nếu cơ sở dữ liệu bị crash, phá hủy tài sản dữ liệu trị giá một triệu đô la vì một số lập trình viên đã làm hỏng phương thức `writeEmployee` đó, giám đốc điều hành cấp C nào sẽ phát hiện ra điều đó đầu tiên? CTO.

Vì vậy, tôi có ba giám đốc điều hành cấp C khác nhau quan tâm đến ba hàm này. Nếu chúng không hoạt động đúng, mỗi người trong số họ sẽ khó chịu vì những lý do khác nhau. Nếu họ muốn thay đổi, nếu các tổ chức cụ thể của họ muốn thay đổi, những thay đổi đó sẽ đến từ bên dưới những cấp C cụ thể đó.

Một thay đổi đối với schema cơ sở dữ liệu sẽ đến từ bên dưới CTO. Một số DBA sẽ muốn thay đổi schema, buộc bạn phải thay đổi lớp này.

Nếu báo cáo cần thay đổi, sẽ có một số kiểm toán viên muốn thay đổi. Anh ta báo cáo với CEO.

Nếu các quy tắc nghiệp vụ để tính lương thay đổi, sẽ có một số kế toán viên làm việc cho CFO.

Vì vậy, lớp này có ba trách nhiệm khác nhau. Nó có trách nhiệm với ba tác nhân khác nhau—ba người khác nhau hoặc ba tổ chức khác nhau—tất cả cùng một lúc.

Có thể tôi thêm một tính năng mới vào hàm `reportHours` và vô tình làm hỏng hàm `calcPay` không? Có thể.

"Ồ, tôi sẽ không làm điều đó. Tôi sẽ không đặt ba phương thức đó vào lớp của mình vì ba phương thức đó có những lý do khác nhau để thay đổi. Chúng thay đổi vào những thời điểm khác nhau vì những lý do khác nhau. Chúng thay đổi dựa trên sự quan tâm của những người khác nhau trong tổ chức. Vì vậy, tôi sẽ tách ba phương thức đó ra và giữ chúng ra khỏi một lớp duy nhất."

Bạn nói, "Khoan đã. Tôi nghĩ đây là các đối tượng. Tôi nghĩ các đối tượng được cho là phải có các phương thức làm cho chúng hoạt động."

Vâng, chắc chắn, điều đó đúng, nhưng bạn cũng nên tự bảo vệ mình khỏi những người cấp C đó, những người sẽ đến và sa thải bạn.

Vậy chúng ta làm gì? Làm thế nào chúng ta đưa ba phương thức đó vào ba lớp khác nhau? Bởi vì tôi không muốn chúng ở đó. Có cả đống cách để làm điều đó.

Bạn có thể làm theo cách này. Bạn có thể tạo một `Employee` có phương thức `calcPay`. Bạn có thể tạo các lớp khác như `ReportWriter` và `EmployeeRepository` mà nói chuyện với phương thức `Employee` và quản lý để thực hiện công việc của chúng. Đó là một khả năng, mặc dù tôi không thích điều đó lắm vì những người này bây giờ phụ thuộc vào cái đó, và nếu có một thay đổi đối với `calcPay`, những người này sẽ phải được thay đổi. Họ sẽ phải được biên dịch lại và triển khai lại. Tôi không quan tâm lắm đến điều đó.

Vì vậy, có lẽ những gì chúng ta sẽ làm là: có lẽ những gì chúng ta sẽ làm là chúng ta sẽ tạo một interface tên là `Employee` có ít nhất một trong những phương thức này, và sau đó chúng ta sẽ kế thừa từ interface đó một lớp khác triển khai `calcPay`. Có lẽ chúng ta có thể làm điều đó.

Hoặc, trời ạ, có lẽ chúng ta có thể có một lớp duy nhất ảnh hưởng đến cả ba phương thức nhưng có ba lớp cơ sở khác nhau, mỗi lớp với một phương thức. Điều đó sẽ thú vị, mặc dù tôi không biết liệu nó có giải quyết vấn đề hay không.

Hoặc có lẽ những gì chúng ta có thể làm là tạo ba lớp, mỗi lớp với một phương thức, và đặt một đối tượng facade ra đây để ủy quyền cho chúng.

Có hàng triệu cách để giải quyết vấn đề này, hàng triệu cách để di chuyển xung quanh nó. Phần quan trọng là: **đừng đặt các hàm thay đổi vì những lý do khác nhau vào cùng một lớp**.

Hàm `reportHours` sẽ thay đổi vì lý do gì? Hàm `reportHours` sẽ thay đổi vì lý do gì? Vâng, họ muốn báo cáo giờ phân số theo một cách khác. Họ không thay đổi nội dung của báo cáo chút nào. Họ chỉ thay đổi định dạng mà họ sử dụng để báo cáo giờ phân số. Họ từng báo cáo giờ phân số bằng dấu thập phân. Bây giờ họ sẽ báo cáo giờ phân số bằng hai số—một cho số giờ, một cho số phần mười—không có dấu chấm nữa. Họ không thích dấu chấm đó.

Bạn nghĩ những người này không làm những điều như vậy sao? Vâng. "Chúng tôi không thích dấu chấm đó." Được rồi.

Vì vậy, bạn làm việc ở đó và bạn thay đổi dấu chấm. Bạn kéo dấu chấm ra, và bạn làm hỏng phương thức `calcPay`. Bạn không muốn các phương thức trong một lớp trong cùng một lớp nếu chúng thay đổi vì những lý do hoàn toàn khác nhau.

Tôi không muốn phơi bày phương thức `calcPay` ra những thay đổi thất thường của các kiểm toán viên muốn thay đổi định dạng của một báo cáo. Định dạng nên đi vào một nơi. Tính toán nên đi vào một nơi khác. Cơ sở dữ liệu ở một nơi khác. Nếu bạn có một số mối quan tâm khác, hãy đặt nó vào một nơi khác. Đừng trộn lẫn các mối quan tâm trong các lớp của bạn.

### O - Nguyên Tắc Mở/Đóng (Open-Closed Principle - OCP)

**Nguyên tắc Mở/Đóng**—chữ O trong SOLID—nhân tiện, được phát minh bởi Bertrand Meyer vào năm 1988 trong một cuốn sách tuyệt vời có tên _Object-Oriented Software Construction_. Nếu bạn chưa đọc cuốn sách đó, hãy đọc nó. Đọc ấn bản đầu tiên. Ấn bản thứ hai dài một nghìn trang. Không ai có thể đọc được nó.

Nguyên tắc Mở/Đóng nói rằng: **một lớp hoặc một module nên mở để mở rộng nhưng đóng để sửa đổi**. Điều đó có nghĩa là gì? Có nghĩa là bạn nên có khả năng thay đổi những gì module làm mà không cần thay đổi module. Bạn nên có khả năng thay đổi hành vi của module mà không cần thay đổi module. Nó nên mở để mở rộng để hành vi của nó có thể được mở rộng, nhưng đóng để sửa đổi.

Bây giờ, điều này nghe có vẻ mâu thuẫn. Nó nghe có vẻ không thể. Làm thế quái nào bạn có thể thay đổi hành vi của một module nếu bạn không thể thay đổi module? Nhưng chúng ta làm điều đó mọi lúc.

Module này đây, module `copy` này, tôi có thể thay đổi hành vi của nó bằng cách viết một driver I/O mới. Tôi có thể viết một driver I/O mới đọc các ký tự từ máy đọc ký tự quang học, và chương trình copy của tôi có thể gọi driver I/O đó. Tôi có phải biên dịch lại chương trình copy của mình không? Không. Tôi không phải thay đổi nó. Tôi không phải biên dịch lại nó. Tôi không phải làm gì với nó cả, và tuy nhiên nó có thể gọi máy đọc ký tự quang học.

Driver máy đọc ký tự quang học được viết lâu sau khi chương trình copy đã được biên dịch và đặt trên máy tính của tôi, nhưng chương trình của tôi vẫn có thể gọi nó. Tôi có thể mở rộng chương trình copy vì tôi có các lời gọi đa hình trong đó, và tôi có thể triển khai các lời gọi đa hình đó bất cứ lúc nào tôi muốn theo bất kỳ cách thú vị nào tôi muốn. Tôi có thể mở rộng chương trình copy.

Nếu bạn có thể tuân thủ nguyên tắc này, thì khi bạn thêm một tính năng mới vào một ứng dụng, bạn nên có khả năng làm điều đó bằng cách viết code mới và không thay đổi bất kỳ code cũ nào. Bạn sẽ không sửa đổi nó. Bạn sẽ mở rộng nó. Vì vậy, tất cả các module của bạn đều đóng để sửa đổi, nhưng chúng có thể được mở rộng, vì vậy bạn có thể mở rộng ứng dụng của mình với một tính năng mới mà không cần thay đổi bất kỳ module hiện có nào.

Nếu bạn đã tuân thủ nguyên tắc đó, điều này có khả thi không? Vâng, điều đó có thể. Tôi có thể làm được. Đây, tôi sẽ chỉ cho bạn. Nó dễ.

#### Phiên Bản Thất Bại (Vi Phạm OCP)

Chúng ta sẽ làm điều này bằng C, hoặc một số ngôn ngữ trông giống như C. Tôi có một ứng dụng đơn giản. Nhân tiện, đây là phiên bản thất bại. Đây là phiên bản không tuân thủ Nguyên tắc Mở/Đóng, được viết bằng C.

Tôi có một lớp `shape.h`. Nó có một enum trong đó. Enum đó có hai enumerator: `CIRCLE` và `SQUARE`. Có một cấu trúc dữ liệu trong đó gọi là `shape` có một phần tử dữ liệu, đó là enumeration.

Tôi có một cấu trúc dữ liệu khác gọi là `circle`. Nó bắt đầu theo cách giống như `shape`, có nghĩa là tôi có thể ép kiểu một `circle` thành một `shape` và vẫn sử dụng nó. Đây là một loại kế thừa giả ở đây. Tôi có một `double radius` và một điểm center, và một hàm có thể vẽ một hình tròn.

```c
// shape.h
enum ShapeType {CIRCLE, SQUARE};

struct Shape {
    enum ShapeType type;
};

// circle.h
struct Circle {
    enum ShapeType type;
    double radius;
    Point center;
};

void drawCircle(struct Circle* c);

// square.h
struct Square {
    enum ShapeType type;
    double side;
    Point topLeft;
};

void drawSquare(struct Square* s);
```

Tôi có một cấu trúc dữ liệu tên là `square`, trong một file khác gọi là `square.h`. Nó cũng bắt đầu với cấu trúc dữ liệu enum, có một `double side` và một điểm `topLeft`, và một hàm vẽ hình vuông.

Bây giờ tôi có hàm này đây, `drawAllShapes`. `drawAllShapes` nhận một mảng các con trỏ shape. Nó lặp qua mảng các con trỏ shape. Nó hỏi mỗi shape kiểu của nó là gì. Nếu đó là một hình vuông, nó gọi `drawSquare`. Nếu đó là một hình tròn, nó gọi `drawCircle`.

```c
void drawAllShapes(struct Shape* shapes[], int count) {
    for (int i = 0; i < count; i++) {
        switch (shapes[i]->type) {
            case SQUARE:
                drawSquare((struct Square*)shapes[i]);
                break;
            case CIRCLE:
                drawCircle((struct Circle*)shapes[i]);
                break;
        }
    }
}
```

Điều tệ gì xảy ra vì tôi viết code theo cách này? Điều này vi phạm Nguyên tắc Mở/Đóng. Vì vậy, nếu tôi thêm một hình oval, tôi phải thêm một switch khác, một case khác vào câu lệnh switch. Nhưng đó không phải là điều đầu tiên tôi làm. Dòng code đầu tiên thay đổi là gì?

Enum. Và đó là trong `shape.h`. Ai `#include` `shape.h`? Mọi người. Ai biên dịch lại? Mọi người. `circle` biên dịch lại vì tôi đã thêm một oval. `square` biên dịch lại vì tôi đã thêm một oval. Điều đó về cơ bản là bệnh hoạn. `square` và `circle` không quan tâm chút nào đến oval, nhưng chúng phải biên dịch lại chỉ vì tôi đã thêm một oval. Có điều gì đó thực sự sai với điều đó.

Đó là cứng nhắc. Đó là triệu chứng của tính cứng nhắc. Bạn phải biên dịch quá nhiều vì một thay đổi—biên dịch những thứ không nên phải được biên dịch lại vì một thay đổi.

Nhưng nó tệ hơn thế vì tôi phải sửa đổi câu lệnh switch. Tôi phải đến câu lệnh switch, và tôi phải thêm case oval. Bạn nghĩ, "Ồ, không tệ lắm. Nó chỉ là một câu lệnh switch." Huh. Nhưng có nhiều hơn một câu lệnh switch. Các câu lệnh switch giống như rận. Luôn có nhiều hơn một. Các câu lệnh switch nhân bản và sao chép và lan truyền qua hệ thống.

Có các câu lệnh switch ở dạng này cho `drawAllShapes`, `eraseAllShapes`, `dragAllShapes`, `scaleAllShapes`, `rotateAllShapes`—bất cứ thứ gì bạn có thể làm với các shape, đều có một câu lệnh switch cho nó. Ngoại trừ việc chúng không phải tất cả đều là câu lệnh switch vì một số lập trình viên không thích câu lệnh switch và họ sử dụng câu lệnh if-else thay thế, nhưng chúng vẫn là câu lệnh switch.

Bây giờ những gì tôi phải làm là tôi phải tìm tất cả chúng, và tôi phải sửa đổi tất cả chúng. Tôi phải đặt oval vào tất cả chúng. Điều đó có vẻ đơn giản, ngoại trừ thực tế là các lập trình viên là những sinh vật kỳ lạ. Họ thích làm các tối ưu hóa logic. Nếu họ có thể loại bỏ một case hoặc làm một số AND và OR đặc biệt hoặc cái gì đó, họ sẽ làm điều đó.

Ví dụ, case của một circle trong `rotateAllShapes` là gì? Không có gì. Bạn không làm gì với một circle khi bạn xoay nó. Nó không có case.

Vì vậy, điều này là dễ vỡ. Nó sẽ vỡ vì tôi sẽ không tìm thấy tất cả các câu lệnh switch đó, và tôi sẽ không giải mã chúng một cách logic đúng đắn, và tôi sẽ kết thúc việc tạo ra một số vấn đề vì tôi đã bỏ lỡ điều gì đó.

Điều này là cứng nhắc vì nó biên dịch lại nhiều hơn mức cần thiết. Nó là dễ vỡ vì tôi không thể tìm thấy tất cả các câu lệnh switch và sửa đổi chúng một cách chính xác.

Nhưng đó không phải là vấn đề tệ nhất. Vấn đề tệ nhất là: sếp của chúng ta đến với chúng ta và nói, "Bạn biết đấy, chúng ta đã gặp rất nhiều rắc rối với công ty khởi nghiệp này, và chúng ta sắp hết vốn. Mô hình bán hàng mà chúng ta có chỉ không hoạt động, nhưng chúng ta có một ý tưởng mới. Chúng ta đã thuê một cố vấn, trả cho anh ta rất nhiều tiền, và anh ta đã nói với chúng ta phải làm gì. Những gì chúng ta phải làm là: chúng ta phải cho `drawAllShapes` miễn phí. Chúng ta sẽ lấy cái đó, chúng ta sẽ đặt nó vào một file JAR hoặc một DLL, chúng ta sẽ đặt nó lên website của chúng ta, cho nó miễn phí. Sau đó chúng ta sẽ lấy tất cả các circle và square và oval và triangle, và chúng ta sẽ đặt chúng vào các file JAR hoặc DLL riêng biệt, và chúng ta sẽ bán tất cả chúng với giá 5 đô la mỗi cái trên website của chúng ta."

Chúng ta phải nói với sếp của mình rằng chúng ta không thể làm điều đó vì câu lệnh switch có một phụ thuộc vào `square` và `circle` và `oval` và `triangle`. Chúng ta không thể tách chúng ra. Chúng ta phải triển khai `circle` và `square` và `triangle` và `oval` với `drawAllShapes`. Không có cách nào để triển khai chúng một cách độc lập.

Chúng ta nói với sếp của mình, "Kiến trúc của chúng ta không hỗ trợ những gì bạn muốn làm." Sau đó chúng ta phá sản và tìm kiếm công việc khác. Đó là **tính bất động** (immobility). Phần có thể tái sử dụng đã không được tách rời đúng cách khỏi phần rối rắm.

#### Phiên Bản OO (Tuân Thủ OCP)

Làm thế nào chúng ta có thể sửa điều đó? Bạn đã sẵn sàng cho "tại sao" chưa? Đây đến rồi. Đây là phiên bản OO. Tôi muốn bạn nghĩ về âm nhạc nhẹ nhàng ở phía sau, âm thanh của gió thổi qua cây, những chú thỏ mờ nhảy qua các ngọn đồi.

```cpp
class Shape {
public:
    virtual void draw() = 0;
};

class Square : public Shape {
public:
    void draw() override {
        // Vẽ hình vuông
    }
};

class Circle : public Shape {
public:
    void draw() override {
        // Vẽ hình tròn
    }
};
```

Và nhìn vào `drawAllShapes`:

```cpp
void drawAllShapes(Shape* shapes[], int count) {
    for (int i = 0; i < count; i++) {
        shapes[i]->draw();
    }
}
```

Sự thanh lịch. Sự vinh quang. Vẻ đẹp của tất cả. Tất cả chúng đều là các shape. Lặp qua các shape và chỉ bảo mỗi cái tự vẽ chính nó.

Ồ, Chúa ơi. Điều gì xảy ra khi chúng ta thêm oval? Ở đây phải biên dịch lại cái gì? Không có gì. Không có gì. Tôi có thể thêm oval. Không có gì ở đây biên dịch lại. Nguyên tắc Mở/Đóng.

Nó có cứng nhắc không? Nó không cứng nhắc. Nó không thể cứng nhắc vì không có gì phải biên dịch lại. Nó có dễ vỡ không? Không thể dễ vỡ vì mọi thứ tôi có thể làm với một shape tôi phải triển khai trong shape đó. Không có cách nào để tôi quên triển khai một phương thức. Nó không dễ vỡ. Tôi không phải đi săn lùng các câu lệnh switch. Không có các tối ưu hóa logic buồn cười. Hàm `rotate` sẽ được triển khai để làm ít nhất một cái gì đó trong `circle`, ngay cả khi nó chỉ là mở-đóng ngoặc nhọn. Nó không dễ vỡ.

Nhưng phần tốt nhất là khi sếp của chúng ta đến với chúng ta và nói, "Vâng, chúng ta muốn đặt các shape đó vào một DLL và chúng ta muốn đặt draw shapes vào một DLL khác," chúng ta có thể làm điều đó. Bởi vì `drawShapes` không có ý tưởng gì rằng square và circle tồn tại. Chính sách cấp cao ở đây không biết về các chi tiết cấp thấp.

Flow của điều khiển vẫn hoạt động theo cách giống nhau. Vòng lặp này gọi các hàm draw này. Nhưng tại thời điểm biên dịch, chính sách cấp cao không biết về các chi tiết cấp thấp. Không có khiếm khuyết thiết kế nào xuất hiện. Không có triệu chứng nào của code tệ là rõ ràng.

#### Lời Nói Dối

Tôi đã nói đây là lời nói dối. Tại sao đây là lời nói dối? Ồ, hóa ra là khách hàng có một ý tưởng khác trong đầu. Điều này bảo vệ chúng ta khỏi điều gì đó. Nó bảo vệ chúng ta khỏi điều gì? Nó bảo vệ chúng ta khỏi những thay đổi nào?

Các shape mới. Nếu chúng ta tạo các shape mới, không có gì ở đây thay đổi. Nhưng khách hàng của chúng ta không quan tâm đến các shape mới. Những gì họ muốn là tất cả các hình vuông được vẽ trước và tất cả các hình tròn được vẽ sau. Đó là thay đổi mà họ muốn thực hiện. Họ muốn tất cả các hình tròn ở trên các hình vuông, vì vậy họ muốn thứ tự thay đổi.

Bây giờ, nếu chúng ta biết điều đó trước, có lẽ chúng ta có thể đã phát minh ra một abstraction bảo vệ chúng ta khỏi thứ tự của các shape. Nhưng chúng ta không biết điều đó trước. Chúng ta nghĩ đây là một mô hình thế giới thực hoàn hảo. Tuyệt vời! Square kế thừa từ shape. Circle kế thừa từ shape. Làm thế nào bạn có thể mô hình hóa thế giới thực tốt hơn điều đó?

Chỉ là đó không phải là những gì khách hàng quan tâm. Khách hàng rất giỏi trong việc bằng cách nào đó biết thiết kế của bạn là gì và sau đó chọn tính năng mới sẽ hoàn toàn phá hoại thiết kế của bạn. Đây là một thực tế của cuộc sống. Đó là một trong những khiếm khuyết lớn của thiết kế OO, bởi vì để thiết kế OO bảo vệ bạn khỏi khách hàng, bạn phải biết khách hàng sẽ làm gì. Khách hàng luôn làm điều khác.

Vì vậy, bạn không thể chỉ mô hình hóa thế giới thực và hy vọng điều tốt nhất vì khách hàng chắc chắn sẽ tìm thấy một số cách thú vị để làm hỏng bạn.

#### Cách Tiếp Cận Thực Dụng

Thay vào đó, những gì chúng ta làm là chúng ta có một quan điểm rất thực dụng. Chúng ta nói, "Được rồi, những gì chúng ta sẽ làm là triển khai thứ đơn giản nhất mà chúng ta có thể. Sau đó chúng ta sẽ đưa nó ra trước mặt khách hàng càng sớm càng tốt. Chúng ta sẽ yêu cầu khách hàng thay đổi nó. Xin hãy thay đổi code của chúng ta."

Bạn đã bao giờ làm điều này chưa? Xin hãy thay đổi code của chúng ta. Hãy cho chúng ta các tính năng mới. Khách hàng sẽ đề xuất các tính năng mới, và điều này cho bạn một manh mối về trục thay đổi thực sự nằm ở đâu trong ứng dụng của bạn. Sau đó bạn có thể triển khai các abstraction bảo vệ bạn khỏi những thay đổi đó.

Đừng nghĩ rằng bạn biết, bởi vì bạn không biết. Hãy ra ngoài với khách hàng càng sớm càng tốt. Tìm hiểu xem họ sẽ làm gì với bạn, và sau đó xây dựng các abstraction vào ứng dụng của bạn sẽ bảo vệ bạn. Chúng sẽ không phải là những gì bạn nghĩ chúng là.

### L - Nguyên Tắc Thay Thế Liskov (Liskov Substitution Principle - LSP)

Tôi nghĩ chúng ta có thời gian cho một nguyên tắc nữa. Nguyên tắc này được gọi là **Nguyên tắc Thay thế Liskov** (Liskov Substitution Principle). Nó được phát minh bởi Barbara Liskov vào năm 1988. 1988 là một năm tốt cho các nguyên tắc.

Barbara Liskov đã nói điều này—ồ, thực ra, đây là một diễn giải vì những gì bà viết là một công thức toán học, nhưng những gì tôi sẽ làm là diễn giải công thức đó: **các lớp kế thừa phải có thể sử dụng được thông qua interface lớp cơ sở mà không cần người dùng phải biết sự khác biệt**.

Điều đó có ý nghĩa hoàn hảo, phải không? Tôi có một user nào đó, và user sử dụng một lớp cơ sở. Nhưng tôi có một lớp kế thừa. Tôi nên có khả năng thay thế lớp kế thừa đó vào, và client không nên biết gì về nó. Đa hình đơn giản, phải không?

#### Vấn Đề Kinh Khủng Rectangle-Square

Tôi có một `Rectangle`. Trong lớp này mà chúng ta đã viết từ lâu, nó có một field tên là `height` và một field tên là `width`. Cả hai đều là số thực. Có một hàm `setHeight` và một hàm `setWidth`.

```java
class Rectangle {
    protected double height;
    protected double width;

    public void setHeight(double h) {
        height = h;
    }

    public void setWidth(double w) {
        width = w;
    }
}
```

Một yêu cầu mới xuất hiện. Yêu cầu mới đó là cho một `Square`. Bây giờ, rõ ràng, một hình vuông là một hình chữ nhật, vì vậy `Square` kế thừa từ `Rectangle`. Hoàn toàn có lý.

Khoan đã. `Rectangle` có bao nhiêu field? Hai: `height` và `width`. `Square` cần bao nhiêu field? Một. Nó sẽ kế thừa bao nhiêu? Có điều gì đó sai. `Square` không thể kế thừa từ `Rectangle` và chỉ có một field.

Bây giờ, điều đó có nghĩa là chúng ta sẽ lãng phí bộ nhớ. Bộ nhớ rẻ. Hãy chỉ lãng phí bộ nhớ và làm điều này. Hãy ghi đè `setWidth` trong `Square` để nó đặt cả height và width. Chúng ta sẽ ghi đè `setHeight` để nó đặt cả height và width. Điều đó giải quyết vấn đề.

```java
class Square extends Rectangle {
    @Override
    public void setWidth(double w) {
        width = w;
        height = w;
    }

    @Override
    public void setHeight(double h) {
        height = h;
        width = h;
    }
}
```

Hãy tưởng tượng một số user của `Rectangle`. User đó gọi `setHeight`. User gọi `setHeight` trên `Rectangle` có quyền mong đợi rằng width sẽ không thay đổi không? Có.

Nhưng nếu tôi truyền cho anh ta một `Square`, anh ta sẽ gọi `setHeight`, và width sẽ thay đổi. Điều đó sẽ làm hỏng máy trạng thái hữu hạn nội bộ của anh ta, và sau đó anh ta sẽ làm hỏng heap. Anh ta sẽ phát hiện ra điều này một tỷ lệnh sau khi bạn nhận được một `NullPointerException`.

Chuyện gì đã xảy ra? Ồ, nhân tiện, bạn là người đang viết code vừa bị nổ. Bạn là người đang viết code đang gọi `setWidth` trên `Rectangle`. Bạn lấy logic analyzer của bạn ra, và bạn đi qua back trace qua một tỷ lệnh. Bạn phát hiện ra, sau nhiều tuần phân tích, rằng lý do bạn bị nổ là vì ai đó đã truyền cho bạn một `Square`.

Vì vậy, bạn sẽ viết code gì trong module của bạn để bảo vệ bạn khỏi khả năng rằng `Rectangle` mà bạn đang giữ thực sự có thể là một `Square`? `if (rectangle instanceof Square)...`

Bằng cách làm như vậy, bạn sẽ treo một phụ thuộc vào `Square`, một thứ mà bạn không bao giờ muốn biết về. Nhưng module của bạn, vốn được cho là biết về rectangle, sẽ đột nhiên có một phụ thuộc vào `Square`, vi phạm Nguyên tắc Mở/Đóng và làm cho bạn cứng nhắc và làm cho bạn dễ vỡ.

Có ai có code như thế trong hệ thống của họ nơi bạn hỏi một đối tượng là kiểu gì không? Chúng ta sử dụng gì trong Java? Chúng ta sử dụng `instanceof`. Trong .NET, chúng ta sử dụng `is` hoặc `as`. Có một loạt các cách khác nhau để làm điều đó.

Không đúng là mỗi lần bạn làm điều đó thì đó là một vấn đề, nhưng hầu hết thời gian khi bạn làm điều đó, đó là một vấn đề. Hầu hết thời gian. Mối quan hệ thú vị cụ thể này gây ra điều đó xảy ra.

#### Điều Gì Đã Sai?

Vậy điều gì đã sai? Một hình vuông không phải là một hình chữ nhật sao? Một hình vuông là một hình chữ nhật, vì vậy đó nên là mối quan hệ đúng.

Bây giờ, vì đó không phải là một hình vuông, đó là một đoạn code. Đó không phải là một hình chữ nhật. Nó là một đoạn code. Đúng là một hình vuông là một hình chữ nhật, nhưng lớp `Square` không phải là một hình vuông. Lớp `Rectangle` không phải là một hình chữ nhật.

Nếu lớp `Square` đại diện cho hình vuông, lớp `Rectangle` đại diện cho một hình chữ nhật. Nhưng **những người đại diện của các thứ không chia sẻ các mối quan hệ của các thứ mà họ đại diện**.

Nếu ai đó ở đây đã li dị, bạn có thể đã có hai luật sư đại diện cho bạn và vợ/chồng của bạn. Hai luật sư đó có lẽ không tự họ đang ly dị. Những thứ đại diện không chia sẻ các mối quan hệ của những thứ mà họ đại diện.

Bạn không thể chỉ đơn giản nói, "Ồ, lớp đó có tên của một cái gì đó, và do đó tôi sẽ sử dụng một mối quan hệ kế thừa." Bạn phải hiểu những mối quan hệ này là gì.

Chúng ta gọi mối quan hệ đó là "is-a" (là một). Đó là cái tên sai cho điều đó. Nó không phải là "is-a." Nó được thừa hưởng từ những người trí tuệ nhân tạo trong những năm 80. Họ đang làm các cấu trúc dữ liệu thú vị gọi là knowledge nets, và họ có các mối quan hệ thú vị: "has-a" (có một), "tastes-like" (nếm giống như), "smells-like" (ngửi giống như), "is-a" (là một). Tất cả họ đều bị sa thải vì nguồn tài trợ của họ cạn kiệt vì trí tuệ nhân tạo không thực sự làm bất cứ điều gì hữu ích. Vì vậy, sau đó tất cả họ trở thành các lập trình viên OO, và họ mang theo các từ viết tắt thú vị của họ. Đó là cách chúng ta có được các mối quan hệ "has-a" và "uses" và "is-a". Họ đặt tên các mối quan hệ này theo cách đó, nhưng chúng là lời nói dối. Chúng không chính xác.

Mối quan hệ kế thừa không phải là mối quan hệ "is-a". Mối quan hệ kế thừa thực sự chỉ là việc khai báo lại các hàm và biến trong một phạm vi con.

#### Cách Sửa Nó

Làm thế nào chúng ta có thể sửa vấn đề này? Mối quan hệ giữa `Square` và `Rectangle` là gì? Họ có... nhưng vâng, bạn muốn như "hành xử như" (behaves like) một thứ. Tôi sẽ làm gì? Tôi sẽ có bất kỳ mối quan hệ nào giữa hai lớp này không? Cả hai có thể kế thừa từ một lớp cơ sở chung. Có lẽ lớp cơ sở là `Shape`. Nhưng hai lớp này thực sự không có gì chung cả. Chúng không có cùng số biến. Chúng không có cùng hành vi. Chúng không liên quan, ít nhất không phải như anh em, và chắc chắn `Rectangle` không phải là cha mẹ của `Square`.

Đây là một vi phạm của Nguyên tắc Thay thế Liskov vì `Square` không thể thay thế cho `Rectangle`, mặc dù nó có vẻ phù hợp với định nghĩa của "is-a". Nó không phù hợp với quy tắc thay thế.

Bất cứ khi nào bạn vi phạm quy tắc thay thế, cuối cùng bạn sẽ thêm một câu lệnh `if` kiểm tra kiểu của đối tượng để ngăn bạn làm crash hệ thống của mình, điều này làm cho... đây là khoảng 8 giờ.

Ồ, bạn muốn tiếp tục? Bạn muốn một phương thức `isRegular`? Bạn muốn cái đó ở trong `Rectangle`? Và những gì bạn đang nói là, "Đó có phải là một hình chữ nhật đều không?" Vì vậy, tôi thậm chí không muốn lớp `Square`. Tôi chỉ muốn một hình chữ nhật đều... Bây giờ điều gì sẽ xảy ra nếu sau đó tôi đặt height của một hình chữ nhật đều? Nó có đặt width không? Tôi có sửa đổi width không? Tôi có dám không? Hay tôi chỉ đặt cờ `isRegular` thành false?

Nếu chúng tình cờ giống nhau, tôi có tự động đặt cờ `isRegular` không? Tôi tính toán nó. Vì vậy `isRegular` sẽ trả về true nếu height và width giống nhau. Okay.

Chúng là số thực. Bạn không thể làm gì với một số thực? Bạn không thể so sánh chúng bằng nhau. Tôi có thể so sánh các số thực bằng nhau không? Mọi người đi tù vì làm điều đó, nhân tiện, đặc biệt nếu số thực đại diện cho tiền. Đừng so sánh các số thực bằng nhau trong máy tính. Các số dấu phẩy động có thể lớn hơn hoặc nhỏ hơn, nhưng chúng không bao giờ bằng nhau. Nếu bạn sử dụng toán tử bằng và nó nói chúng bằng nhau, đó là lời nói dối.

Tôi không thể so sánh chúng bằng nhau. Bạn có thể so sánh chúng gần, nhưng bạn không thể so sánh chúng bằng nhau.

## Kết Luận

Được rồi, tôi nghĩ chúng ta đã xong cho ngày hôm nay. Cảm ơn tất cả các bạn đã chú ý. Nếu có bất kỳ câu hỏi nào, tôi sẽ ở lại một lúc, nhưng phần còn lại của các bạn có thể đi.
